# Open Resource ID format

An Open Resource Id, or "orid" for short, is a semi-flexible identifier format to uniquely identify resources in a cloud-like or multi-cloud environment. An ORID, sometimes referred to as an "ORID key", is a plain text delimited string where each element, or segment, has a specific meaning to the consumer of the ORID.

## General / Common Requirements

* All ORID keys are delimited by a colon, `:`, character.
* All ORID keys start with the `orid` segment.
* The second segment of a ORID is always the version of the ORID specification the ORID adheres to.
* Segments of a ORID will be blank or empty string if they are not needed.
    * Ex: `orid:1::::foo-service:bar-type/baz-resource` is a valid ORID key where the three custom fields are not utilized.

## ORID consuming service considerations

* Services should be expected to safely ignore custom segments on an ORID unless the service has been configured by an organizations operator to utilize these fields.
    * Ex: The `custom-1` field may represent an "account" that a specific `resource-id` resides under. An operator may configure the service to check a security token provided by the caller in a HTTP/HTTPS request and validate that the caller has access to perform actions against the account provided in the `custom-1` segment.

## Version 1 (Oct 2019)

The first version of was created with single provider concerns in mind. This can be thought of as all services or resources that reside inside of a single cloud provider such as AWS, GCP or Microsoft Azure.

### Examples

* orid:1:`custom-1`:`custom-2`:`custom-3`:`service`:`resource-id`
* orid:1:`custom-1`:`custom-2`:`custom-3`:`service`:`resource-type/resource-id`
* orid:1:`custom-1`:`custom-2`:`custom-3`:`service`:`resource-type`:`resource-id`

### Definitions

* `custom-1`, `custom-2` and `custom-3` are variable and should be defined by the consumer of the ORID. An organization that consumes ORIDs should make every effort to make these elements consistent across their infrastructure / systems.
    * Example usages may be to include a organization specific sub version number for `custom-1` while using `custom-2` to identify that a service or resource resides in a specific data center within the organization.
* `service` is the system that will act upon the resource defined in the next segment/segments
* `resource-id` is a unique identifier for some entity inside of the referenced service. One example would be if the *service* were some sort of Functions as a Service platform the *resource-id* would indicate a specific function.
* `resource-type/resource-id` and `resource-type:resource-id` can be used in leu of just `service:resource-id` in the use case that a simple `resource-id` is not enough. One example would be if the *service* were some kind of storage mechanism where the `resource-id` defined a specific element of data inside the container identified by the `resource-type` segment. Another example may be a message queue implementation described in the example below.
    * Storage example: `orid1::::storage-solution:container-123:file-abc` or `orid1::::storage-solution:container-123/file-abc`.
    * Queue example: `orid1::::queue-solution:queue-id:message-id` or `orid1::::queue-solution:queue-id/message-id`.