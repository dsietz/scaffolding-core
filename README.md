[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Docs.rs](https://docs.rs/scaffolding-core/badge.svg)](https://docs.rs/scaffolding-core)
![Build/Test](https://github.com/dsietz/scaffolding-core/actions/workflows/master.yaml/badge.svg)
[![Discussions](https://img.shields.io/github/discussions/dsietz/scaffolding-core)](https://github.com/dsietz/scaffolding-core/discussions)

# Scaffolding Core

For software development teams who appreciate a kick-start to their object oriented development, this scaffolding library is a light-weight module that provides the basic structures and behavior that is the building block of all class intantiated objects. Unlike the practice of writing classes with the various approaches for building common functionality, this open source library helps you inherit these cross-class commonalities so you can focus on the differentiator that define your class.   

---

### Table of Contents
- [Scaffolding Core](#scaffolding-core)
    - [Table of Contents](#table-of-contents)
  - [What's New](#whats-new)
  - [Usage](#usage)
      - [Addresses](#addresses)
      - [Email Addresses](#email-addresses)
      - [Metadata](#metadata)
      - [Notes](#notes)
      - [Phone Numbers](#phone-numbers)
      - [Tagging](#tagging)
  - [How to Contribute](#how-to-contribute)
  - [License](#license)

---

## What's New
We made the crate easier to implement and updated the documentation and the crate metadata.

**2.0.0**
+ [re-export dependent crates for ease of easier usability](https://github.com/dsietz/scaffolding-core/issues/45)
+ [Clean up](https://github.com/dsietz/scaffolding-core/issues/46)

## Usage

(1) Import the necessary modules
```rust
extern crate scaffolding_core;

use scaffolding_core::*;
```
(2) Add Scaffolding attributes and apply the trait to a `struct` 
```rust
#[scaffolding_struct]
#[derive(Debug, Clone, Deserialize, Serialize, Scaffolding)]
struct MyEntity {
    a: bool,
    b: String,
}
```
(3) Dynamically add the default values for the Scaffodling attributes as part of the `impl` `::new()` constructor
_NOTE:_ The `scaffolding_core::defaults` module is available if you wish to use it else where
```rust
impl MyEntity {
    // Define the constructor - Optional
    // Note: Any of the Scaffolding attributes that are set here 
    //       will not be overwritten when generated. For example
    //       the `id` attribute, if uncommented, would be ignored.
    #[scaffolding_fn]
    fn new(arg: bool) -> Self {
        let msg = format!("You said it is {}", arg);
        Self {
            // id: "my unique identitifer".to_string(),
            a: arg,
            b: msg
        }
    }

    fn my_func(&self) -> String {
        "my function".to_string()
    }
}
```
(4) Use the Scaffolding attributes and behavior
```rust
let mut entity = MyEntity::new(true);

/* scaffolding attributes */
assert_eq!(entity.id.len(), "54324f57-9e6b-4142-b68d-1d4c86572d0a".len());
assert_eq!(entity.created_dtm, defaults::now());
assert_eq!(entity.modified_dtm, defaults::now());
// becomes inactive in 90 days
assert_eq!(entity.inactive_dtm, defaults::add_days(defaults::now(), 90));
// expires in 3 years
assert_eq!(entity.expired_dtm, defaults::add_years(defaults::now(), 3));

/* use the activity log functionality  */
// (1) Log an activity
entity.log_activity("cancelled".to_string(), "The customer has cancelled their service".to_string());
// (2) Get activities
assert_eq!(entity.get_activity("cancelled".to_string()).len(), 1);

/* custom attributes */
assert_eq!(entity.a, true);
assert_eq!(entity.b, "You said it is true");

/* custom behavior */
assert_eq!(entity.my_func(), "my function");
```
__Serializing__
```rust
let json_string = entity.serialize();
println!("{}", json_string);
```
__Deserializing__
```rust
let json = r#"{
    "id":"b4d6c6db-7468-400a-8536-a5e83b1f2bdc",
    "created_dtm":1711802687,
    "modified_dtm":1711802687,
    "inactive_dtm":1719578687,
    "expired_dtm":1806410687,
    "activity":[
        {
            "created_dtm":1711802687,
            "action":"updated",
            "description":"The object has been updated"
        },
        {
            "created_dtm":1711802687,
            "action":"updated",
            "description":"The object has been updated"
        },
        {
            "created_dtm":1711802687,
            "action":"cancelled",
            "description":"The object has been cancelled"
        }
        ]
    }"#;
let entity = MyEntity::deserialized(json.as_bytes()).unwrap();

assert_eq!(entity.get_activity("cancelled".to_string()).len(), 1);
```
---
There are additional Scaffolding features that can be applied.

#### Addresses
```rust
#[scaffolding_struct("addresses")]
#[derive(Debug, Clone, Deserialize, Serialize, Scaffolding, ScaffoldingAddresses)]
struct MyEntity {}

impl MyEntity {
    #[scaffolding_fn("addresses")]
    fn new() -> Self {
        Self {}
    }
}

let mut entity = MyEntity::new();

/* use the addresses functionality */
// (1) Add an address
let id = entity.add_address(
    "shipping".to_string(),
    "acmes company".to_string(),
    "14 Main Street".to_string(),
    "Big City, NY 038845".to_string(),
    "USA".to_string(),
    "USA".to_string(),
);
// (2) Find addresses based on the category
let shipping_addresses = entity.addresses_by_category("shipping".to_string());
// (3) Remove an address
entity.remove_address(id);
```
#### Email Addresses
```rust
#[scaffolding_struct("email_addresses")]
#[derive(Debug, Clone, Deserialize, Serialize, Scaffolding, ScaffoldingEmailAddresses)]
struct MyEntity {}

impl MyEntity {
    #[scaffolding_fn("email_addresses")]
    fn new() -> Self {
        Self {}
    }
}

let mut entity = MyEntity::new();

/* use the email addresses functionality */
// (1) Add an email address
let id = entity.insert_email_address(
    "home".to_string(),
    "myemail@example.com".to_string(),
);
// (2) Find email addresses based on the category
let home_email_addresses = entity.search_email_addresses_by_category("home".to_string());
// (3) Remove an address
entity.remove_address(id);
```
#### Metadata
```rust
#[scaffolding_struct("metadata")]
#[derive(Debug, Clone, Deserialize, Serialize, Scaffolding)]
struct MyEntity {}

impl MyEntity {
    #[scaffolding_fn("metadata")]
    fn new() -> Self {
        Self {}
    }
}

let mut entity = MyEntity::new();

/* use the metadata functionality
   Note: `memtadata` is a BTreeMap<String, String>
          https://doc.rust-lang.org/std/collections/struct.BTreeMap.html
*/
entity.metadata.insert("field_1".to_string(), "myvalue".to_string());
assert_eq!(entity.metadata.len(), 1);
```
#### Notes
```rust
#[scaffolding_struct("notes")]
#[derive(Debug, Clone, Deserialize, Serialize, Scaffolding, ScaffoldingNotes)]
struct MyEntity {}

impl MyEntity {
    #[scaffolding_fn("notes")]
    fn new() -> Self {
        Self {}
    }
}

let mut entity = MyEntity::new();

// (1) Insert a note
let id = entity.insert_note(
  "fsmith".to_string(),
  "This was updated".as_bytes().to_vec(),
  None,
);
// (2) Modify the note
entity.modify_note(
  id.clone(),
  "fsmith".to_string(),
  "This was updated again".as_bytes().to_vec(),
  Some("private".to_string()),
);
// (3) Read the note's content
let read_note = entity.get_note(id.clone()).unwrap().content_as_string().unwrap();
println!("{}", read_note);
// (4) Search for notes that contain the word `updated`
let search_results = entity.search_notes("updated".to_string());
assert_eq!(search_results.len(), 1);
// (5) Delete the note
entity.remove_note(id);
```
#### Phone Numbers
```rust
#[scaffolding_struct("phone_numbers")]
#[derive(Debug, Clone, Deserialize, Serialize, Scaffolding, ScaffoldingPhoneNumbers)]
struct MyEntity {}

impl MyEntity {
    #[scaffolding_fn("phone_numbers")]
    fn new() -> Self {
        Self {}
    }
}

let mut entity = MyEntity::new();

/* use the phone number functionality */
// (1) Add a phone number
let _ = entity.add_phone_number(
    "home".to_string(),
    "8482493561".to_string(),
    "USA".to_string(),
);
let id = entity.add_phone_number(
    "work".to_string(),
    "2223330000".to_string(),
    "USA".to_string(),
);
// (2) Find phone number based on the category
let home_phone = entity.phone_numbers_by_category("home".to_string());
// (3) Remove an address
entity.remove_phone_number(id);
```
#### Tagging
```rust
#[scaffolding_struct("tags")]
#[derive(Debug, Clone, Deserialize, Serialize, Scaffolding, ScaffoldingTags)]
struct MyEntity {}

impl MyEntity {
    #[scaffolding_fn("tags")]
    fn new() -> Self {
        Self {}
    }
}

let mut entity = MyEntity::new();

// manage tags
entity.add_tag("tag_1".to_string());
entity.add_tag("tag_2".to_string());
entity.add_tag("tag_3".to_string());
assert!(entity.has_tag("tag_1".to_string()));
entity.remove_tag("tag_2".to_string());
assert_eq!(entity.tags.len(), 2);
```

## How to Contribute

Details on how to contribute can be found in the [CONTRIBUTING](./CONTRIBUTING.md) file.

## License

The `scaffolding-core` project is primarily distributed under the terms of the Apache License (Version 2.0).

See [LICENSE-APACHE "Apache License](./LICENSE-APACHE) for details.