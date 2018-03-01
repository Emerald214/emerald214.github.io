This project is discontinued. It's been moved to http://codinglegend.blog/

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
  <version>2.0-rc1</version>
</dependency>
<dependency>
  <groupId>info.magnolia.rest</groupId>
  <artifactId>magnolia-rest-content-delivery</artifactId>
  <version>2.0-rc1</version>
</dependency>
```

## Registering the delivery endpoint

REST endpoints can be configured in YAML, following the typical light module conventions.  
The Content Delivery module does _not_ provide a default endpoint config; it rather lets you configure yours, with minimal overhead.

## configuration

Delivery endpoint supports two ways for configuring muliple delivery endpoints:

#### 1) By system file path

File path: `<light-module-name>/restEndpoints/<path-to-endpoint>/<endpoint-name>.yaml`

```yaml
class: info.magnolia.rest.delivery.jcr.v2.JcrDeliveryEndpointDefinition
implementationClass: info.magnolia.rest.delivery.jcr.v2.JcrDeliveryEndpoint
workspace: website
rootPath: /travel
nodeTypes:
  - mgnl:page
childNodeTypes:
  - mgnl:page
  - mgnl:area
  - mgnl:component
depth: 1
includeSystemProperties: false
bypassWorkspaceAcls: true
```

For example:

_.../modules/awesomeEndpointModule/restEndpoints/path/to/awesomeEndpoint.yaml_
_.../modules/lovelyEndpointModule/restEndpoints/route/to/lovelyEndpoint.yaml_

#### 2) By `endpointPath`

File path: `<light-module-name>/restEndpoints/<endpoint-name>.yaml`

```yaml
class: info.magnolia.rest.delivery.jcr.v2.JcrDeliveryEndpointDefinition
implementationClass: info.magnolia.rest.delivery.jcr.v2.JcrDeliveryEndpoint
workspace: website
endpointPath: /path/to/endpoint
rootPath: /travel
nodeTypes:
  - mgnl:page
childNodeTypes:
  - mgnl:page
  - mgnl:area
  - mgnl:component
depth: 1
includeSystemProperties: false
bypassWorkspaceAcls: true
```

#### Parameters

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

## Reading a single node

```
GET /{endpoint-path}/{resource-path}
```

| Path parameters   | Description |
| ----------------- | ----------- |
| `endpoint-path` | An endpoint path depending on how you configure in [Link to Header](#configuration) |
| `resource-path` | A node path relative to the workspace root, or configured `rootPath`  |

##### Example

Given the [YAML configuration above](#yaml-configuration), we can make the following request

```sh
curl --request GET \
  --url http://<domain>/.rest/delivery/pages/v1/about
```

##### Response

```json
{
    "@name": "about",
    "@path": "/travel/about",
    "hideInNav": "false",
    "title": "About",
    "main": {
        "@name": "main",
        "@path": "/travel/about/main",
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

## Delivery endpoint v1 (Deprecated)

⚠️ Mind that, despite allowing config of multiple endpoints in different YAML files and/or JCR config,
**there may be only one registration per endpoint class** (bound to its annotated `@Path`).  
This holds true for the `JcrDeliveryEndpoint`.

In case multiple endpoints with same class are configured, it is unspecified which one RESTEasy will pick;
unregistering one may also unregister all of them.

### YAML Configuration

The `JcrDeliveryEndpointDefinition` is essentially a map of endpoint prefixes to target workspaces with various settings.

Config path: `<light module name>/restEndpoints/<endpoint name>.yaml`

```yaml
class: info.magnolia.rest.delivery.jcr.v1.JcrDeliveryEndpointDefinition
params:
  pages:
    workspace: website
    rootPath: /travel
    nodeTypes:
      - mgnl:page
    childNodeTypes:
      - mgnl:page
      - mgnl:area
      - mgnl:component
    depth: 1
    includeSystemProperties: false
    bypassWorkspaceAcls: true
  stories:
    includeSystemProperties: false
    ...
```

| Name                                                                                                                | Required | Description | Default | 
| ------------------------------------------------------------------------------------------------------------------- | -------- | ----------- | ------- |
| `<endpoint name>`                                                                                                   | -        | The endpoint's registered name | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`class`                                                                               | Yes      | | `info.magnolia.rest.delivery.jcr.v1.JcrDeliveryEndpointDefinition` |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`implementationClass`                                                                 | No       | | `info.magnolia.rest.delivery.jcr.v1.JcrDeliveryEndpoint` |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`params`                                                                              | Yes      | Defines the target workspaces and settings to expose, based on endpoint prefixes.<br/>If no param entry is configured, then no workspace is exposed through the delivery endpoint. | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`<endpoint-prefix>`                                     | Yes      | Defines an endpoint prefix where requests will be routed and handled according to the associated workspace parameters below | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`workspace`               | No       | Defines the target [workspace](https://documentation.magnolia-cms.com/display/DOCS/Workspaces) to serve content from. | If empty, the endpoint prefix is used |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`rootPath`                | No       | Defines the root path used to resolve the given node path parameter, and to execute queries. | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`depth`                   | No       | Defines the depth for child-nodes to be included in the responses. | `0` |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`includeSystemProperties` | No       | Defines whether jcr:—and mgnl:-prefixed properties will be included in responses. | `true` |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`nodeTypes`               | No       | List of [primary node types](https://documentation.magnolia-cms.com/display/DOCS/Node+types#Nodetypes-Magnoliaprimarynodetypes) for filtering queried node | `mgnl:content` |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`childNodeTypes`          | No       | List of [primary node types](https://documentation.magnolia-cms.com/display/DOCS/Node+types#Nodetypes-Magnoliaprimarynodetypes) for filtering child nodes | `mgnl:contentNode` |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`limit`                   | No       | Defines the amount of results to return in a paginated result set. | `10` |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`bypassWorkspaceAcls`     | No       | Defines whether or not workspace permissions (ACLs) should be evaluated.<br/>URI permissions are still evaluated. | `false` |


### Reading a single node

```
GET /delivery/{endpoint-prefix}/v1/{path}
```

| Path parameters   | Description |
| ----------------- | ----------- |
| `endpoint-prefix` | A configured endpoint-prefix (the key in the definition `params` map) |
| `path`            | A node path relative to the workspace root, or configured `rootPath`  |

##### Example

Given the [YAML configuration above](#yaml-configuration), we can make the following request

```sh
curl --request GET \
  --url http://<domain>/.rest/delivery/pages/v1/about
```

##### Response

```json
{
    "@name": "about",
    "@path": "/travel/about",
    "hideInNav": "false",
    "title": "About",
    "main": {
        "@name": "main",
        "@path": "/travel/about/main",
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

### Querying nodes

If no path within the workspace is given, the delivery endpoint builds and executes a JCR query.

```
GET /delivery/{endpoint-prefix}/v1?key1=value1&key2=value2
```

| Path parameters   | Description |
| ----------------- | ----------- |
| `endpoint-prefix` | A configured endpoint-prefix (the key in the definition `params` map) |


| Query parameters  | Description | Value |
| ----------------- | ----------- | ----- |
| `q`               | Search terms for full-text search | |
| `orderBy`         | One or multiple property names and directions to sort the query | for example `mgnl:lastModified desc,title asc` |
| `offset`          | The start index in a paginated result set | defaults to `0` |
| `limit`           | The amount of results to return in a paginated result set | defaults to `10` |

#### Listing and paging

##### Example

Given the [YAML configuration above](#yaml-configuration), we can make the following request

```sh
curl --request GET \
  --url http://<domain>/.rest/delivery/pages/v1?limit=2
```

##### Response
```json
{
    "results": [
        {
            "@name": "about",
            "@path": "/travel/about",
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
#### Full-text search

##### Example

```sh
curl --request GET \
  --url http://<domain>/.rest/delivery/stories/v1?q=Zermatt&limit=2
```

##### Response

```json
{
    "results": [
        {
            "@name": "0",
            "@path": "/stories-demo/lost-and-found-in-swiss-alps/0",
            "text": "<h2>Zermatt</h2>",
            "@nodes": []
        },
        {
            "@name": "1",
            "@path": "/stories-demo/lost-and-found-in-swiss-alps/1",
            "text": "<p>I started off in the late morning with the train from Bern to Zermatt. I had a hangover from four beers the previous night - simply drank too much after those nights of abstinence while being on-call. All my gear was stashed in a large backpack, and a group of Swiss asked me if I was going to the Matterhorn.</p>",
            "@nodes": []
        }
    ]
}
```

#### Filters

* Filter has this format `property[operator]=value`
  * For example, _http://<domain>/.rest/delivery/pages/v1?title[like]=tour_
* Supported operators include
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
  * For example,  _http://<domain>/.rest/delivery/pages/v1?mgnl:created[in]=2018-01-01~2018-02-01_
* For filtering by time, only two these formats of ISO 8601 are accepted 
  * Date: yyyy-MM-dd
  * Datetime: yyyy-MM-dd'T'HH:mm:ss.SSSXXX
  * For example,
    * _2018-01-01_
    * _2018-01-11T10:26:47.438+07:00_
* If no operator is provided, `eq` is used by default
* Mind that tbe brackets should be URL-encoded for some tools

##### Example

```sh
curl --request GET \
  --url http://<domain>/.rest/delivery/pages/v1?title=Our+Company
```

##### Response

```json
{
    "results": [
        {
            "@name": "company",
            "@path": "/travel/about/company",
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

##### Special properties for filters

* `@name`: Filter nodes by their node name
* `@ancestor`: Filter nodes under a given path


### Expand references

A node may contain references to other nodes. With the `references` property you can extend the configuration to let it resolve the referenced nodes in other workspaces.

```yaml
class: info.magnolia.rest.delivery.jcr.JcrDeliveryEndpointDefinition
params:
  tours:
    rootPath: /magnolia-travels
    includeSystemProperties: false
    references:
      - propertyName: tourType
        referenceResolver:
          implementationClass: info.magnolia.rest.reference.jcr.UuidReferenceResolver
          targetWorkspace: category
```

| Name                                                                                         | Required | Description | Default | 
| -------------------------------------------------------------------------------------------- | -------- | ----------- | ------- |
|`references`                                                                                  | No       | Defines the list of references to resolve and expand | |
|&nbsp;&nbsp;&nbsp;&nbsp;`-`                                                                   | -        | A reference definition | `info.magnolia.rest.reference.ReferenceDefinition` |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`propertyName`                                | Yes      | Defines for which property to resolve a reference. This supports regular expressions. | |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`nodeType`                                    | No       | Defines which node type to apply the reference to | |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`referenceResolver`                           | No       | A reference resolver definition | `info.magnolia.rest.reference.ReferenceResolverDefinition` |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`implementationClass` | Yes      | Defines the reference resolver implementation to use.<br/>must extend from `info.magnolia.rest.reference.ReferenceResolver` | The following UUID-based resolver is provided for convenience:<br/> `info.magnolia.rest.reference.jcr.UuidReferenceResolver` |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`targetWorkspace`     | Yes      | Defines within which target workspace the resolver should operate | |

##### Example1

```sh
curl --request GET \
  --url http://<domain>/.rest/delivery/tours/v1/Temples-of-the-Orient
```

##### Response

```json
{
    "@name": "Temples-of-the-Orient",
    "@path": "/magnolia-travels/Temples-of-the-Orient",
    "body": "...",
    "name": "Temples of the Orient",
    "description": "Go deep into the spiritual heart of a continent",
    "destination": [
        "7ec72c48-c33f-418e-b2ff-44cfb4bbb1f2"
    ],
    "location": "Beijing, China",
    "duration": "21",
    "author": "Magnolia Travels",
    /* a property named "tourType": "e007e401-1bf8-4658-8293-b9c743784264" would be expanded as follows: */ 
    "tourType": {
        "@name": "tourType",
        "@path": "/tour-types/cultural",
        "body": "...",
        "name": "cultural",
        "level": "level-1",
        "description": "Experiences of a lifetime",
        "icon": "jcr:58c420b6-fa30-4578-8af0-c062ba51e5fb",
        "displayName": "Cultural",
        "image": "jcr:a792164f-5a2a-4708-b3c8-62b48a578200",
        "@nodes": []
    },
    "image": "jcr:9269e62f-cb98-47eb-91ce-db1f09040896",
    "@nodes": [
        "tourType"
    ]
}
```

##### Example2:
Using IN operator

```sh
curl --request GET \
--url http://<domain>/.rest/delivery/pages/v1?mgnl:created%5Bin%5D=2018-01-01~2018-02-01
```
The above URI comes from _http://<domain>/.rest/delivery/pages/v1?mgnl:created[in]=2018-01-01~2018-02-01_ with brackets URL-encoded.

##### Response

```json
{
  "results": [
    {
      "@id": "a8f82d85-bb21-4563-88fa-bbbe4f3ca2f3",
      "@name": "tourFinder",
      "@nodeType": "mgnl:page",
      "@nodes": [
        "main",
        "footer"
      ],
      "@path": "/travel/tourFinder",
      "footer": {
        "@id": "07dbe8c0-5791-4c28-86d6-18cbe233d8b5",
        "@name": "footer",
        "@nodeType": "mgnl:area",
        "@nodes": [],
        "@path": "/travel/tourFinder/footer"
      },
      "hideInNav": "false",
      "main": {
        "@id": "7525daab-a7b1-4a49-9ed0-81b2a0e493bb",
        "@name": "main",
        "@nodeType": "mgnl:area",
        "@nodes": [],
        "@path": "/travel/tourFinder/main"
      }
    }
  ]
}
```

### Exception Handling

Exceptions are caught in exception mappers and the responses are structurally displayed in requested media type
* If requested media type is not supported, JSON is returned as fallback
* If no resource method is matched, JSON is returned as fallback
* Legacy endpoints no longer always return exception in plain text

The tables below show several common exceptions, their status code and error code respectively.

##### JAX-RS Exceptions

| Exception              | HTTP status code | Error code       | Description                           | 
|------------------------|------------------|------------------|---------------------------------------| 
| NotAuthorizedException | 401              | notAuthorized    | Unauthorized                          | 
| BadRequestException    | 400              | badRequest       | Bad request                           | 
| NotAllowedException    | 405              | methodNotAllowed | Request method is not supported       | 
| NotAcceptableException | 406              | notAcceptable    | Requested media type is not supported | 
| NotFoundException      | 404              | notFound         | Resource not found                    | 
| ReaderException        | 400              | readerError      | JAX-RS exception                      | 
| WriterException        | 500              | writerError      | JAX-RS exception                      | 

##### Repository Exceptions

| Exception                | HTTP status code | Error code         | Description                           | 
|--------------------------|------------------|--------------------|---------------------------------------| 
| AccessDeniedException    | 403              | accessDenied       | Access is denied                      | 
| NoSuchNodeTypeException  | 500              | noSuchNodeType     | Requested node type is not found      | 
| NoSuchWorkspaceException | 500              | noSuchWorkspace    | Requested workspace is not configured | 
| ValueFormatException     | 400              | invalidValueFormat | Value format is invalid               | 
| InvalidQueryException    | 400              | invalidQuery       | Query is invalid                      | 
| PathNotFoundException    | 404              | pathNotFound       | Requested resource path is not found  | 

##### Unknown exceptions

As for the exceptions that are not mentioned above, the status code is 500 and the error code is "unknown".

##### Example

```sh
curl --request GET \
  --url 'http://<domain>/.rest/delivery/endpointPrefixNotConfigured/v1'
```

##### Response

```
{
  "error": {
    "code": "notFound",
    "message": "No workspace-params entry for endpoint prefix 'endpointPrefixNotConfigured'"
  }
}
```

### Known limitations

* There may be only one `JcrDeliveryEndpointDefinition` registered.
* Reference links are not supported yet.
* References to the dam workspace are not supported yet.
  * meanwhile, dam links are easy to deduce - be sure to check supported paths with `info.magnolia.dam.core.download.DamDownloadServlet`
