# Messaging API

## API Specification Format

An API specification is organized in files, where a single file represents a resource provided
on the Messaging API.

### Resource

A resource is identified by its name and it has a list of exposed methods and events.

The top-level structure of the API specification file is the following:

```
<full_resource_name>/<method_name>:
  <method_spec>

<full_resource_name>/<method_name>:
  <method_spec>

<full_resource_name>#<event_type>:
  <event_spec>

```

Where `<full_resource_name>` is a fully qualified resource name (including namespaces/parent resource names etc.). For example:
```
accounting.transactions
accounts
```

`<method_name>` is a name as valid identifier (`[A-Za-z_][A-Za-z0-9_]*`).

`<method_spec>` is a specification of the method, described further below

`<event_type>` is a name as valid identifier (`[A-Za-z_][A-Za-z0-9_]*`). As by convention the naming of Resource methods and events follows different rules (verb vs past participle), and different delimiters in targets (`\` vs `#`), so
there should be no name clashes.

`<event_spec>` is a specification of the event, described further below

An example of `accounts.yml`:
```yaml
accounts/show:
  return:
    id: :uid16
    revision: :integer
    json: :string
    signatures:
      :array: :string
    created_at: :timestamp
    updated_at: :timestamp

accounts/update:
  params:
    reference:
      -
      - :string
    description:
      -
      - :string
  return:
    id: :uid16
    reference: :string
    description: :string
    balance: :decimal
    created_at: :timestamp
    updated_at: :timestamp


accounts#updated:
    ...
```

### Method

```
<full_resource_name>/<method_name>:
  [params: <params_spec>]
  [return: <return_spec>]
```

A method is an API endpoint of the Resource that can accept Query or Command. It consists of a name, a set of accepted parameters and a response.

#### Method parameters

`params` object defines a set of parameters that the method accepts. If it is missing or empty, the method has no documented/required accepted parameters.
`params` object is seen as a map between parameter names and `<param_spec>` defining the parameter properties such as data type and default value.
```
...
params:
    <param_name>: <param_spec>
   [<param_name>: <param_spec>
    ...]
```

`<param_spec>` is either a `<type_spec>` defining an accepted data type of the parameter, or an empty value, representing that any value is accepted.

`<type_spec>` can be any of:
 * empty value
 * literal value (string or integer)
 * type reference (Symbol)
 * an array of `<type_spec>`, representing a union of types
 * an Object, in which case it is a `<typedef_spec>`


 `<typedef_spec>` is used to define new types, using Object. By default `<typedef_spec>` describes an Object based type, where its properties have corresponding `<type_spec>` definitions. However, if there is any of the keys `:array` or `:string` (etc) among the properties, the described type is not an object and depends on the key.

 TODO: expand base types, keys: :integer?


Example:
```yaml
...
params:
  id:              # empty <param_spec> indicates that any value is accepted
  name: :string    # data type as a type reference
  value:           # data type as a union
    -              # null literal represents a null as accepted value, i.e. parameter is optional
    - :integer
  state:           # another data type as a union
    - requested    # note that it is a string literal,
    - processed    # it represents the actual accepted value
    - failed
  details:         # data type as an object with following properties
    tx_id: :string
    label:         # an optional string parameter
      -
      - :string
  signatures:      # a type defining an array of strings
    :array: :string
```

#### Method response

`return` key in the `<method_spec>` defines the method response.

If a method specification has a `return` key, the method can both be invoked by Query and Command. If the `return` key is missing, the method can only be invoked by Command.

`<return_spec>` can be any of:

* empty value, indicating the response can be anything
* data type as a type reference, where type is always an Object
* an array, indicating a union of data types that can be returned
* an Object, defining properties of the response and the corresponding `<type_spec>`

Examples:
```yaml
...
# methods:
stats:
  return: # empty <return_spec> indicates that response can be anything

show:
  return:
    id: :uid16   # data type as type reference
    value:       # any value can be returned

list:
  return:
    list:        # an array of 'accounting.transactions'
      :array: :accounting.transactions
    pagination:
      -          # it is either empty, or an object
      - page: :integer
        page_count: :integer
        per_page: :integer
        total_count: :integer


```

## Data Types

Whenever there is a string literal starting with `:` on the right side of the definition, it indicates a type reference, e.g. `:string`, `:integer` etc.

There is a number of built-in data types, as well as a way to define custom data types.

### Built-in data types

#### :string

Represents any string value. E.g. `"Hello, world"`.

#### :integer

Represents any integer value. No limit on the range of possible values is assumed. E.g. `0`, `42`, `-65535`.

#### :uid16

Represents a standard unique 16 bytes identifier value. It is a string of 32 hexadecimal lowercase digits, e.g.: `"3814f58b21f576c5e5040fca83cd2248"`

#### :decimal

Represents a decimal value, formatted as a string with a decimal point form -- long form, as opposed to scientific (exponential) notation. Imposes no assumptions on allowed value range or precision. E.g. `"0"`, `"100000.23"`,
`"3.14159265358979"`, `"-0.0013294"`.

#### :timestamp

Represents a date-time value formatted as a string conforming ISO 8601 in UTC timezone. E.g.: `"2018-05-24T17:16:44.880Z"`

#### :boolean

Represents a boolean value, `true` or `false`.

#### :array

Represents an array of undetermined values.

#### :object

Represents an object with undetermined properties.


### Defining custom data types

It is possible to define a custom data type by specifing top-level key starting with `:` as the type name:
```
:<type_name>: <typedef_spec>
```

For example:
```yaml
:accounting.transactions:
  id: :uid16
  revision: :integer
  from_account_id: :uid16
  to_account_id: :uid16
  json: :string
  signatures: :array_of_signatures # references another custom data type
  created_at: :timestamp
  updated_at: :timestamp

:array_of_signatures:
  :array: :string
```

## Error Reporting

Any application MAY respond to a QUERY request with an error. In that case, the response should comply
with the [Error Reporting](Error_Reporting.md) specification.




