# Magnolia REST Content Delivery

The JCR Delivery endpoint serves JCR content in a concise JSON format.  
Content may be pages, components, stories or anything else that is stored in a workspace.

It offers two methods for consuming content:

* Reading a single node, by passing a specific node path
* Querying for nodes; passing query parameters to leverage pagination, sorting, full-text search as well as filters

The endpoint behavior can be configured with a `JcrDeliveryEndpointDefinition` to match specific workspaces or node types.
Nodes are represented in the JSON output as plain object-graph, resembling the tree-structure of JCR nodes and properties.  
Additionally, UUID references to other workspaces can be resolved and expanded within returned records.

## Getting started

First, install the *rest-content-delivery* module, possibly via Maven; it also depends upon Magnolia's *rest-integration* module.

```xml
<dependency>
  <groupId>info.magnolia.rest</groupId>
  <artifactId>magnolia-rest-integration</artifactId>
  <version>2.1</version>
</dependency>
<dependency>
  <groupId>info.magnolia.rest</groupId>
  <artifactId>magnolia-rest-content-delivery</artifactId>
  <version>2.1</version>
</dependency>
```

## Registering the delivery endpoint

REST endpoints can be configured in YAML, following the typical light module conventions.  
The Content Delivery module does _not_ provide a default endpoint config; it rather lets you configure yours, with minimal overhead.

## YAML configuration

Content Delivery endpoints can be dynamically configured in [light modules](https://documentation.magnolia-cms.com/display/DOCS56/Definition+decoration#Definitiondecoration-Definitiondecoratorfilelocation), typically for each type of content. Magnolia's REST module uses `Convention over Configuration` to generate the API base path, etc.

### 1) File path

`<light-module>/restEndpoints/<path>/<to>/<endpoint-name>_<version-tag>.yaml`

| File path (under /restEndpoints)    | Endpoint path                  |
| -----------------------------------| ------------------------------ |
| `/stories.yaml`					    | `/.rest/stories`               |
| `/delivery/stories.yaml`		        | `/.rest/delivery/stories`      |
| `/stories/v3.yaml` (1)				  | `/.rest/stories/v3`            |
| `/stories_v3.yaml` (2)				  | `/.rest/stories/v3`            |
| `/delivery/stories_v3.yaml `  		  | `/.rest/delivery/stories/v3`   |

* (1) Example 3rd revision of an endpoint definition
* (2) Alternative supported syntax to keep a meaningful name

⚠️ Make sure you have web access on the endpoint path.

Please refer to [DynamicPath](https://git.magnolia-cms.com/projects/MODULES/repos/rest/browse/magnolia-rest-integration/src/main/java/info/magnolia/rest/DynamicPath.java) for technical details and take a look at [best practices](#3-best-practices) to configure it properly.

### 2) File content

```yaml
class: info.magnolia.rest.delivery.jcr.v2.JcrDeliveryEndpointDefinition
workspace: website
rootPath: /travel
nodeTypes:
  - mgnl:page
childNodeTypes:
  - mgnl:page
  - mgnl:area
  - mgnl:component
depth: 1
bypassWorkspaceAcls: true
includeSystemProperties: false
```

| Name                      | Required | Description | Default | 
| ------------------------- | -------- | ----------- | ------- |
| `class`                   | Yes      | | `info.magnolia.rest.delivery.jcr.v2.JcrDeliveryEndpointDefinition` |
| `implementationClass`     | No       | | `info.magnolia.rest.delivery.jcr.v2.JcrDeliveryEndpoint` |
| `workspace`               | No       | Defines the target [workspace](https://documentation.magnolia-cms.com/display/DOCS/Workspaces) to serve content from. | If empty, the endpoint prefix is used |
| `endpointPath`            | No       | Defines the endpoint path and ignore the system file path. | |
| `rootPath`                | No       | Defines the root path used to resolve the given node path parameter, and to execute queries. | |
| `depth`                   | No       | Defines the depth for child-nodes to be included in the responses. | `0` |
| `includeSystemProperties` | No       | Defines whether jcr:—and mgnl:-prefixed properties will be included in responses. | `true` |
| `nodeTypes`               | No       | List of [primary node types](https://documentation.magnolia-cms.com/display/DOCS/Node+types#Nodetypes-Magnoliaprimarynodetypes) for filtering queried node | `mgnl:content` |
| `childNodeTypes`          | No       | List of [primary node types](https://documentation.magnolia-cms.com/display/DOCS/Node+types#Nodetypes-Magnoliaprimarynodetypes) for filtering child nodes | `mgnl:contentNode` |
| `limit`                   | No       | Defines the amount of results to return in a paginated result set. | `10` |
| `bypassWorkspaceAcls`     | No       | Defines whether or not workspace permissions (ACLs) should be evaluated.<br/>URI permissions are still evaluated. | `false` |

Lastly, if the endpoint path convention does not fit, the `endpointPath` property can be configured, and then overrides the automatic path.

### 3) Best practices

* You are now in control of your API versioning strategy
  * Versioning the API with the version suffix, for best consistency, can be combined with YAML deprecations on Magnolia 5.6
  * Don't version the API, keep it evolving in sync with rapidly changing data models, templates and frontend apps
  * Our declarative API versioning is now decoupled from versioning of the endpoint implementation
* Our previous `v1.JcrDeliveryEndpoint` was statically mapped to `/.rest/delivery/<endpoint-prefix>/v1`. Magnolia's security setup comes with the `rest-anonymous` role, which grants public GET access to `/.rest/delivery/*`.
  * If you want to leverage Magnolia's default setup for anonymous access, you may now place your endpoint definitions inside a `delivery` folder to match this pattern
  * If not, ensure appropriate URI web access in Roles configuration

## Reading a single node

```
GET /{endpoint-path}/{node-path}
```

| Path parameters   | Description |
| ----------------- | ----------- |
| `endpoint-path`   | An endpoint path depending on how you configure in [YAML configuration](#yaml-configuration) |
| `node-path`       | A node path relative to the workspace root, or configured `rootPath`  |

##### Example

Given the [YAML configuration above](#yaml-configuration), we can make the following request

```sh
curl --request GET \
  --url http://<host>/.rest/delivery/pages/about
```

##### Response

```json
{
    "@name": "about",
    "@path": "/travel/about",
    "@id": "808ebe4c-72b2-49f1-b9f7-e7db22bce02f",
    "@nodeType": "mgnl:page",
    "hideInNav": "false",
    "title": "About",
    "main": {
        "@name": "main",
        "@path": "/travel/about/main",
        "@id": "f3b2681f-e747-4ff6-bcbb-2a9a9f01553e",
        "@nodeType": "mgnl:area",
        "@nodes": []
    },
    "footer": { ... },
    "company": { ... },
    "what-we-believe": { ... },
    "careers": { ... },
    "@nodes": [
        "main",
        "footer",
        "company",
        "what-we-believe",
        "careers"
    ]
}
```

## Querying nodes

If no `<node-path>` is given, the delivery endpoint builds and executes a JCR query.

```
GET /{endpoint-path}?param1=value1&param2=value2&...
```

| Path parameters   | Description |
| ----------------- | ----------- |
| `endpoint-path`   | A configured endpoint path |


| Query parameters  | Description | Value | Default |
| ----------------- | ----------- | ----- | ------- |
| `q`               | Search terms for full-text search | | 
| `orderBy`         | One or multiple property names and directions to sort the query | for example `mgnl:lastModified desc,title asc` | If no direction is specified, `asc` is used. |
| `offset`          | The start index in a paginated result set | defaults to `0` | |
| `limit`           | The amount of results to return in a paginated result set | defaults to `10` | |

* Mind that if no `orderBy` is passed, the order depends on `respectDocumentOrder` config of JackRabbit

### Listing and paging

##### Example

Given the [YAML configuration above](#yaml-configuration), we can make the following request

```sh
curl --request GET \
  --url http://<host>/.rest/delivery/pages?limit=2
```

##### Response
```json
{
    "results": [
        {
            "@name": "about",
            "@path": "/travel/about",
            "@id": "808ebe4c-72b2-49f1-b9f7-e7db22bce02f",
            "@nodeType": "mgnl:page",
            "hideInNav": "false",
            "title": "About",
            "main": { ... },
            "footer": { ... },
            "company": { ... },
            "what-we-believe": { ... },
            "careers": { ... },
            "@nodes": [
                "main",
                "footer",
                "company",
                "what-we-believe",
                "careers"
            ]
        },
        {
            "@name": "about-demo",
            "@path": "/travel/meta/about-demo",
            "@id": "c1e9ac13-60c1-430a-8ee3-6c70078bf403",
            "@nodeType": "mgnl:page",
            "hideInNav": "false",
            "title": "About this demo",
            "main": { ... },
            "footer": { ... },
            "@nodes": [
                "main",
                "footer"
            ]
        }
    ]
}
```
### Full-text search

##### Example

```sh
curl --request GET \
  --url http://<host>/.rest/delivery/pages?q=marketing
```

##### Response

```json
{
    "results": [
        {
            "@name": "marketing-associate",
            "@path": "/travel/about/careers/marketing-associate",
            "@id": "f19c60f2-3049-4883-a170-4bf65e3abb91",
            "@nodeType": "mgnl:page",
            "hideInNav": "false",
            "title": "Marketing Associate",
            "main": { ... },
            "footer": { ... },
            "@nodes": [
                "main",
                "footer"
            ]
        }
    ]
}
```

### Filters

* Filter has this format `property[operator]=value`
  * For example, `http://<host>/.rest/delivery/pages?title[like]=tour`
* Supported operators
  * `ne` means `<>`
  * `eq` means `=`
  * `lt` means `<`
  * `lte` means `<=`
  * `gt` means `>`
  * `gte` means `>=`
  * `like` means `LIKE`
  * `in` means `IN`
  * `not-in` means `NOT IN`
* For `in` and `not-in`, a range symbol `~` should be provided
  * For example,  `http://<host>/.rest/delivery/pages?mgnl:created[in]=2018-01-01~2018-02-01`
* For filtering by time, only two ISO-8601-based formats are accepted 
  * Date: `yyyy-MM-dd`
    * For example, `2018-01-01`
  * Datetime: `yyyy-MM-dd'T'HH:mm:ss.SSSXXX`
    * For example, `2018-01-11T10:26:47.438+07:00`
* If no operator is provided, `eq` is used by default
* Mind that tbe brackets should be URL-encoded for some tools

#### Special properties for filters

* `@name`: Filter nodes by their node name
* `@ancestor`: Filter nodes under a given path

##### Example1

```sh
curl --request GET \
  --url http://<host>/.rest/delivery/pages?title=Our+Company
```

##### Response

```json
{
    "results": [
        {
            "@name": "company",
            "@path": "/travel/about/company",
            "@id": "8fa4a73f-51c3-40ac-b698-715595216186",
            "@nodeType": "mgnl:page",
            "hideInNav": "false",
            "title": "Our Company",
            "main": { ... },
            "footer": { ... },
            "@nodes": [
                "main",
                "footer"
            ]
        }
    ]
}
```

##### Example2:
Using IN operator

```sh
curl --request GET --globoff \
 --url 'http://<host>/.rest/delivery/pages?mgnl:created[in]=2018-01-01~2018-02-01'
```

##### Response

```json
{
    "results": [
        {
            "@name": "stories",
            "@path": "/travel/stories",
            "@id": "f8cbedd6-df91-4d7c-a952-2dc414d97704",
            "@nodeType": "mgnl:page",
            "title": "Stories",
            "story": {
                "@name": "story",
                "@path": "/travel/stories/story",
                "@id": "a096ec20-c190-4b2e-91a4-294c99b64049",
                "@nodeType": "mgnl:page",
                "hideInNav": "true",
                "title": "Story",
                "@nodes": []
            },
            "main": {
                "@name": "main",
                "@path": "/travel/stories/main",
                "@id": "22ba6501-a0fa-4762-bb6c-37cb729166df",
                "@nodeType": "mgnl:area",
                "@nodes": []
            },
            "footer": {
                "@name": "footer",
                "@path": "/travel/stories/footer",
                "@id": "ca3dd2d9-6660-43e5-b580-9063de9ca60e",
                "@nodeType": "mgnl:area",
                "@nodes": []
            },
            "@nodes": [
                "story",
                "main",
                "footer"
            ]
        }
    ]
}
```

## Expand references

A node may contain references to other nodes. With the `references` property you can extend the configuration to let it resolve the referenced nodes in other workspaces.

```yaml
class: info.magnolia.rest.delivery.jcr.v2.JcrDeliveryEndpointDefinition
workspace: tours
bypassWorkspaceAcls: true
includeSystemProperties: false
references:
  - name: tourTypeReference
    propertyName: tourTypes
    referenceResolver:
      class: info.magnolia.rest.reference.jcr.JcrReferenceResolverDefinition
      targetWorkspace: category
      expand: true
      generateLink: true
```

| Name                                                                                         | Required | Description | Default | 
| -------------------------------------------------------------------------------------------- | -------- | ----------- | ------- |
|`references`                                                                                  | No       | Defines the list of references to resolve and expand | |
|&nbsp;&nbsp;&nbsp;&nbsp;`-`                                                                   | -        | A reference definition | `info.magnolia.rest.reference.ReferenceDefinition` |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`propertyName`                                | Yes      | Defines for which property to resolve a reference. This supports regular expressions. | |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`nodeType`                                    | No       | Defines which node type to apply the reference to | |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`referenceResolver`                           | No       | A reference resolver definition | `info.magnolia.rest.reference.ReferenceResolverDefinition` |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`class`               | Yes      | Defines the reference resolver class to use.<br/>It must extend from `info.magnolia.rest.reference.ReferenceResolverDefinition` | The following resolver is provided for convenience:<br/>`info.magnolia.rest.reference.jcr.JcrReferenceResolverDefinition` |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`implementationClass` | No       | Defines the reference resolver implementation to use.<br/>It must extend from `info.magnolia.rest.reference.ReferenceResolverDefinition` | The following resolver is provided for convenience:<br/>`info.magnolia.rest.reference.jcr.ConfiguredJcrReferenceResolverDefinition` |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`targetWorkspace`     | Yes      | Defines within which target workspace the resolver should operate | |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`generateLink`        | No       | A flag to show the link to node | false |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`expand`              | No       | A flag to expand the object | true |

##### Example

```sh
curl --request GET \
  --url http://<host>/.rest/delivery/tours/magnolia-travels/Hut-to-Hut-in-the-Swiss-Alps
```

##### Response

```json
{
    "@name": "Hut-to-Hut-in-the-Swiss-Alps",
    "@path": "/magnolia-travels/Hut-to-Hut-in-the-Swiss-Alps",
    "@id": "d30b9ca6-e22d-45c5-9456-7538ab7bccf8",
    "@nodeType": "mgnl:content",
    "name": "Hut to Hut in the Swiss Alps",
    "description": "...",
    "location": "Zurich, Switzerland",
    "tourTypes": [
        {
            "@name": "active",
            "@path": "/tour-types/active",
            "@id": "eaf9a648-fae1-48ae-a293-69bed874f159",
            "@nodeType": "mgnl:category",
            "body_de": "...",
            "body": "...",
            "description_de": "...",
            "name": "active",
            "level": "level-1",
            "description": "...",
            "displayName_de": "Aktiv",
            "icon": "jcr:57ad1ee6-87a5-4fb7-ac8c-16f6fdf35d28",
            "displayName": "Active",
            "image": "jcr:b601ef57-a44a-432d-bc9a-bb228890b01d",
            "@link": "http://localhost:8080/magnolia/tour-types/active",
            "@nodes": []
        }
    ],
    "author": "Magnolia Travels",
    "body": "...",
    "destination": [
        "6cc50e28-fb0e-4e49-b3b6-728690a2e861"
    ],
    "duration": "7",
    "image": "jcr:80d091ae-22ce-4cfc-b302-cb39f25c3d30",
    "@nodes": []
}
```

### Expand asset

You can also expand an asset using `AssetReferenceResolverDefinition`

```
class: info.magnolia.rest.delivery.jcr.v2.JcrDeliveryEndpointDefinition
workspace: tours
bypassWorkspaceAcls: true
includeSystemProperties: false
references:
  - name: tourImageReference
    propertyName: image
    referenceResolver:
      class: info.magnolia.rest.reference.dam.AssetReferenceResolverDefinition
```

| Name                                                                                          | Required | Description | Default | 
| --------------------------------------------------------------------------------------------- | -------- | ----------- | ------- |
| \<Same as above\>                                                                             |          |             |         |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`referenceResolver`                            | No       | A reference resolver definition | `info.magnolia.rest.reference.ReferenceResolverDefinition` |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`class`                | Yes      | Defines the reference resolver class to use.<br/>It must extend from `info.magnolia.rest.reference.ReferenceResolverDefinition` | The following resolver is provided for convenience:<br/>`info.magnolia.rest.reference.dam.AssetReferenceResolverDefinition` |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`implementationClass`  | No       | Defines the reference resolver implementation to use.<br/>It must extend from `info.magnolia.rest.reference.ConfiguredReferenceResolverDefinition` | The following resolver is provided for convenience:<br/>`info.magnolia.rest.reference.dam.ConfiguredAssetReferenceResolverDefinition` |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`expand`               | No       | A flag to expand the object | true |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`includeAssetMetadata` | No       | A flag to include asset meta data | true |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`includeDownloadLink`  | No       | A flag to include download link | true |

##### Example

```sh
curl --request GET \
  --url http://<host>/.rest/delivery/tours/magnolia-travels/Hut-to-Hut-in-the-Swiss-Alps
```

##### Response
```json
{
    "@name": "Hut-to-Hut-in-the-Swiss-Alps",
    "@path": "/magnolia-travels/Hut-to-Hut-in-the-Swiss-Alps",
    "@id": "d30b9ca6-e22d-45c5-9456-7538ab7bccf8",
    "@nodeType": "mgnl:content",
    "name": "Hut to Hut in the Swiss Alps",
    "description": "...",
    "location": "Zurich, Switzerland",
    "tourTypes": [
        "eaf9a648-fae1-48ae-a293-69bed874f159"
    ],
    "author": "Magnolia Travels",
    "body": "...",
    "destination": [
        "6cc50e28-fb0e-4e49-b3b6-728690a2e861"
    ],
    "duration": "7",
    "image": {
        "@name": "flickr-swiss-trails-ed-coyle-3797048134_1f4a930f48_o.jpg",
        "@path": "/tours/flickr-swiss-trails-ed-coyle-3797048134_1f4a930f48_o.jpg",
        "@id": "jcr:80d091ae-22ce-4cfc-b302-cb39f25c3d30",
        "@link": "/magnoliaAuthor/dam/jcr:80d091ae-22ce-4cfc-b302-cb39f25c3d30/flickr-swiss-trails-ed-coyle-3797048134_1f4a930f48_o.jpg",
        "metadata": {
            "fileName": "flickr-swiss-trails-ed-coyle-3797048134_1f4a930f48_o.jpg",
            "mimeType": "image/jpeg",
            "caption": "Ed Coyle",
            "fileSize": "910566",
            "height": "1361",
            "width": "2048",
            "format": "image/jpeg",
            "rights": "by-nd/2.0",
            "creator": [
                "superuser"
            ],
            "date": "2015-06-03T14:32:41.867+07:00",
            "created": "2015-01-29T14:54:01.034+07:00",
            "modified": "2015-06-03T14:32:41.867+07:00"
        }
    },
    "@nodes": []
}
```

## Exception handling

Exceptions are caught in exception mappers and the responses are structurally displayed in requested media type
* If requested media type is not supported, JSON is returned as fallback
* If no resource method is matched, JSON is returned as fallback
* Legacy endpoints no longer always return exception in plain text

The tables below show several common exceptions, their status code and error code respectively.

### JAX-RS Exceptions

| Exception              | HTTP status code | Error code       | Description                           | 
|------------------------|------------------|------------------|---------------------------------------| 
| NotAuthorizedException | 401              | notAuthorized    | Unauthorized                          | 
| BadRequestException    | 400              | badRequest       | Bad request                           | 
| NotAllowedException    | 405              | methodNotAllowed | Request method is not supported       | 
| NotAcceptableException | 406              | notAcceptable    | Requested media type is not supported | 
| NotFoundException      | 404              | notFound         | Resource not found                    | 
| ReaderException        | 400              | readerError      | JAX-RS exception                      | 
| WriterException        | 500              | writerError      | JAX-RS exception                      | 

### Repository Exceptions

| Exception                | HTTP status code | Error code         | Description                           | 
|--------------------------|------------------|--------------------|---------------------------------------| 
| AccessDeniedException    | 403              | accessDenied       | Access is denied                      | 
| NoSuchNodeTypeException  | 500              | noSuchNodeType     | Requested node type is not found      | 
| NoSuchWorkspaceException | 500              | noSuchWorkspace    | Requested workspace is not configured | 
| ValueFormatException     | 400              | invalidValueFormat | Value format is invalid               | 
| InvalidQueryException    | 400              | invalidQuery       | Query is invalid                      | 
| PathNotFoundException    | 404              | pathNotFound       | Requested resource path is not found  | 

##### Example

```sh
curl --request GET \
  --url 'http://<host>/.rest/delivery/pages/pathNoFound'
```

##### Response

```
{
    "error": {
        "code": "pathNotFound",
        "message": "/pathNoFound"
    }
}
```

##### Unknown exceptions

As for the exceptions that are not mentioned above, the status code is 500 and the error code is "unknown".

## Localization

Users can get localized content by using either `lang` request parameter or `Accept-Language` HTTP header.

* If both are provided, `lang` has higher priority.
* If both are not provided, the locale is retrieved by `18nContentSupport#getLocale`

### 1) `lang` request parameter

* The languages are configured in `i18n` node of the corresponding site
  * For example, `/travel/i18n` for `travel` site
* `lang=all` returns data without i18n effect
* If no locale matches with requested locale, the closest supported locale is returned
  * See `AbstractI18nContentSupport#getNextLocale`

##### Example:

```sh
curl --request GET --url http://<host>/.rest/delivery/pages/about?lang=de
```

##### Response

```
{
    "@name": "about",
    "@path": "/travel/about",
    "@id": "808ebe4c-72b2-49f1-b9f7-e7db22bce02f",
    "@nodeType": "mgnl:page",
    "hideInNav": "false",
    "title": "Über uns",
    "main": { ... },
    "footer": { ... },
    "company": {
        "@name": "company",
        "@path": "/travel/about/company",
        "@id": "8fa4a73f-51c3-40ac-b698-715595216186",
        "@nodeType": "mgnl:page",
        "hideInNav": "false",
        "title": "Unser Unternehmen",
        "@nodes": []
    },
    "what-we-believe": {
        "@name": "what-we-believe",
        "@path": "/travel/about/what-we-believe",
        "@id": "f28c5cec-3567-485e-a8db-196339aa82b5",
        "@nodeType": "mgnl:page",
        "hideInNav": "false",
        "title": "Woran wir glauben",
        "@nodes": []
    },
    "careers": {
        "@name": "careers",
        "@path": "/travel/about/careers",
        "@id": "30058d95-f1c8-4b0a-b0ad-34b710e7c367",
        "@nodeType": "mgnl:page",
        "hideInNav": "false",
        "title": "Karriere",
        "@nodes": []
    },
    "@nodes": [
        "main",
        "footer",
        "company",
        "what-we-believe",
        "careers"
    ]
}
```


### 2) `Accept-Language` request HTTP header 

Similar to [Accept-Language](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language)

* `*` wildcard
  * For example:  `Accept-Language: *`
  * It accepts any language sever returns
* Multiple types weighted by `quality value`
  * For example, `fr-CH, fr;q=0.9, en;q=0.8, de;q=0.7, *;q=0.5`
  * Best match of supported languages is selected based on weight

##### Example

```
curl --request GET --url http://<host>/.rest/delivery/pages/about --header "Accept-Language: fr;q=0.9, de;q=0.8, en;q=0.7"
```

##### Response

Same response with 1)
