# Dataclasses Avro Schema Generator

Generate [Avro](https://avro.apache.org/docs/1.8.2/spec.html) Schemas from a Python class

[![Build Status](https://travis-ci.org/marcosschroh/dataclasses-avroschema.svg?branch=master)](https://travis-ci.org//dataclasses-avroschema)
[![GitHub license](https://img.shields.io/github/license/marcosschroh/dataclasses-avroschema.svg)](https://github.com/marcosschroh/dataclasses-avroschema/blob/master/LICENSE)
[![codecov](https://codecov.io/gh/marcosschroh/dataclasses-avroschema/branch/master/graph/badge.svg)](https://codecov.io/gh/marcosschroh/dataclasses-avroschema)
![python version](https://img.shields.io/badge/python-3.7%2B-yellowgreen)

## Requirements

`python 3.7+`

## Installation

```bash
pip install dataclasses-avroschema
```

## Documentation

https://marcosschroh.github.io/dataclasses-avroschema/

## Usage

### Generating the avro schema

```python
import typing

from dataclasses_avroschema import AvroModel, types


class User(AvroModel):
    "An User"
    name: str
    age: int
    pets: typing.List[str]
    accounts: typing.Dict[str, int]
    favorite_colors: types.Enum = types.Enum(["BLUE", "YELLOW", "GREEN"])
    country: str = "Argentina"
    address: str = None

    class Meta:
        namespace = "User.v1"
        aliases = ["user-v1", "super user"]

User.avro_schema()

'{
    "type": "record",
    "name": "User",
    "doc": "An User",
    "namespace": "User.v1",
    "aliases": ["user-v1", "super user"],
    "fields": [
        {"name": "name", "type": "string"},
        {"name": "age", "type": "int"},
        {"name": "pets", "type": "array", "items": "string"},
        {"name": "accounts", "type": "map", "values": "int"},
        {"name": "favorite_colors", "type": "enum", "symbols": ["BLUE", "YELLOW", "GREEN"]},
        {"name": "country", "type": ["string", "null"], "default": "Argentina"},
        {"name": "address", "type": ["null", "string"], "default": "null"}
    ]
}'
```

### Serialization to avro or avro-json and json payload

For serialization is neccesary to use python class/dataclasses instance

```python
from dataclasses import dataclass

import typing

from dataclasses_avroschema import AvroModel


@dataclass
class Address(AvroModel):
    "An Address"
    street: str
    street_number: int

@dataclass
class User(AvroModel):
    "User with multiple Address"
    name: str
    age: int
    addresses: typing.List[Address]

address_data = {
    "street": "test",
    "street_number": 10,
}

# create an Address instance
address = Address(**address_data)

data_user = {
    "name": "john",
    "age": 20,
    "addresses": [address],
}

# create an User instance
user = User(**data_user)

user.serialize()
# >>> b"\x08john(\x02\x08test\x14\x00"

user.serialize(serialization_type="avro-json")
# >>> b'{"name": "john", "age": 20, "addresses": [{"street": "test", "street_number": 10}]}'

# Get the json from the instance

user.to_json()
# python dict >>> {'name': 'john', 'age': 20, 'addresses': [{'street': 'test', 'street_number': 10}]}

```

### Deserialization

Deserialization could take place with an instance dataclass or the dataclass itself

```python
import typing

from dataclasses_avroschema import AvroModel


class Address(AvroModel):
    "An Address"
    street: str
    street_number: int

class User(AvroModel):
    "User with multiple Address"
    name: str
    age: int
    addresses: typing.List[Address]

avro_binary = b"\x08john(\x02\x08test\x14\x00"
avro_json_binary = b'{"name": "john", "age": 20, "addresses": [{"street": "test", "street_number": 10}]}'

User.deserialize(avro_binary)
# >>> {"name": "john", "age": 20, "addresses": [{"street": "test", "street_number": 10}]}

User.deserialize(avro_json_binary, serialization_type="avro-json")
# >>> {"name": "john", "age": 20, "addresses": [{"street": "test", "street_number": 10}]}
```

## Features

* [X] Primitive types: int, long, float, boolean, string and null support
* [X] Complex types: enum, array, map, fixed, unions and records support
* [x] Logical Types: date, time, datetime, uuid support
* [X] Schema relations (oneToOne, oneToMany)
* [X] Recursive Schemas
* [X] Generate Avro Schemas from `faust.Record`
* [X] Instance serialization correspondent to `avro schema` generated
* [X] Data deserialization
* [X] Generate json from python class instance

## Development

1. Create a `virtualenv`: `python3.7 -m venv venv && source venv/bin/activate`
2. Install requirements: `pip install -r requirements.txt`
3. Code linting: `./scripts/lint`
4. Run tests: `./scripts/test`
