# Error Reporting

Whenever an application responds to a QUERY request, it MAY respond with an error message.

An error message is identified by the presence of the key `error`:
```
{
  "error": "This is an error, not a successful response"
}
```

Additionally, an application MAY include a `code` attribute and `params` object:

* `code` if present, contains a machine readable code describing the error condition,
for example that the requested resource is not found

* `params` is an Object containing some attributes, which may have special meaning depending
on the value of the `code`.

If the `code` attribute is missing, the error message shoud be treated as an unspecified error, a rough equivalent of
"HTTP 500 Internal server error" response of a Web application.

The following is a specification of an error message:
```yaml
error_message:
  error: :string   # human readable error message
  code:
    - # optional
    - :string      # machine readable error code
  params:
    - # optional
    - :object      # an Object containing additional error related data, specific to code
```

## Error Codes

`code`, if present, contains a string identifier:

Code         | Description
-------------|-----------------
not_found    | The requested resource is not found
invalid      | The QUERY request is not valid

Additionally, applications MAY respond with custom `code` values, different from the listed ones.
If the application that receives the error message does not recognize these custom codes, it
should treat the error message as unspecified error.

### Code: not_found

Normally, when an application provides a Messaging API to some resource, it exposes `show` method
expecting the resource ID.

If the resource with the given ID is not found, the application MUST respond with an error message
having the code `not_found` and an empty `params` attribute.

```
> QUERY "accounts/show" {"id":"00000000000000000000000000000000"}
<

{
  "error": "Resource not found",
  "code": "not_found"
}

```

### Code: invalid

Whenever an application receiving a QUERY request finds out that some or all attributes of the
request are invalid, it MUST respond with an error message having the code `invalid` and `params`
object containing invalid request attributes with corresponding Attribute Error Codes.

The value of the `params` attribute is an Object with following structure:
```
attribute1: <array of Attribute Error Codes>
attribute2: <array of Attribute Error Codes>
...
```

Only attributes with non-empty list of error codes are expected to be specified in `params`.


### Attribute Error Codes

Attribute Error Code is associated with a particular attribute of the request and describes a reason
(or reasons) why this attribute has been considered invalid;

Available error codes:

Code               |  Description
-------------------|----------------------
blank              | Required attribute is blank
invalid            | Attribute value is invalid (e.g. invalid format, or the value is not within range etc)
taken              | Unique attribute value is already taken
not_found          | Object referenced by the attribute is not found


For example, whenever we issue a request to create or update some resource, there are certain constraints
on the passed parameters:

```
> QUERY "customers/create" {"name":null}
<

{
  "error": "Resource is invalid",
  "code": "invalid",
  "params": {
    "name": ["blank"]
  }
}
```


## See Also

* [error_message.yml](error_message.yml) -- Formal error message specification
