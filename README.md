# Content Delivery Endpoint
Content Delivery Endpoint allows you to get content through a RESTful API. The nodes can be pages, components, stories or anything else that is stored in a named workspace. The response is displayed in JSON format.

## How to set up
Make sure you have "rest-content-delivery" module installed then create the following configuration

### YAML configuration
Config path: ```rest-services/restEndpoints/<endpoint name>.yaml```
```yaml
class: info.magnolia.rest.delivery.jcr.ConfiguredJcrDeliveryEndpointDefinition
implementationClass: info.magnolia.rest.delivery.jcr.v1.JcrDeliveryEndpoint
params:
  website:
    depth: 1
    rootPath: /travel
    includeSystemProperties: false
    nodeTypes:
      0: mgnl:page
      1: mgnl:area
      2: mgnl:component
  stories:
    includeSystemProperties: false
```

| Name | Requisiteness | Description | Default | 
| ------ | ------ | ------ | ------ |
| ```<endpoint nane>``` | Yes | Endpoint name | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```class``` | Yes | info.magnolia.rest.delivery.jcr.ConfiguredJcrDeliveryEndpointDefinition | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```implementationClass``` | Yes | info.magnolia.rest.delivery.jcr.v1.JcrDeliveryEndpoint | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```params``` | Yes | Contains the workspaces you want to expose and deliver | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```<workspace name>``` | Yes | Name of [workspace](https://documentation.magnolia-cms.com/display/DOCS/Workspaces) | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```depth``` | No | Depth of child nodes to include. 0 is just the node. 1 is node and its children etc. | 0 |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```includeSystemProperties``` | No | Whether system properties are included | true |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```nodeTypes``` | No | List of [primary node types](https://documentation.magnolia-cms.com/display/DOCS/Node+types#Nodetypes-Magnoliaprimarynodetypes) | mgnl:content, mgnl:contentNode |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```rootPath``` | No | Path configured as the root of the workspace. Only content below the path is requested. | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```limit``` | No | The number of nodes displayed in a page | 10 |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```bypassWorkspaceAcls``` | No | Allow bypassing workspace ACLs | false |

## How to use

There are two ways to get node data
* Reading node: Specify node path and get that node in hierarchical form
* Querying nodes: Pass query params and get a list of nodes. This way has some advantages:
  * Listing
  * Full-text search
  * Filtering

### Reading node

```
GET /delivery/{workspace}/v1/{path}
```

| Parameter | Default value | Description | Parameter type | Data type | 
| ------ | ------ | ------ | ------ | ------ |
| ```workspace``` | | [workspace](https://documentation.magnolia-cms.com/display/DOCS/Workspaces) | path | string |
| ```path``` | | Path to node | path | string |

#### Example
Given [the YAML configuration](#yaml-configuration), we can make the following request
```sh
curl -u superuser:superuser "http://<yourDomain>/.rest/delivery/website/v1/about"
```

#### Response
```json
{
    "@name": "about",
    "@path": "/travel/about",
    "hideInNav": "false",
    "title": "About",
    "title_de": "Über uns",
    "main": {
        "@name": "main",
        "@path": "/travel/about/main",
        "@nodes": []
    },
    "footer": {
        "@name": "footer",
        "@path": "/travel/about/footer",
        "@nodes": []
    },
    "company": {
        "@name": "company",
        "@path": "/travel/about/company",
        "hideInNav": "false",
        "title": "Our Company",
        "title_de": "Unser Unternehmen",
        "@nodes": []
    },
    "what-we-believe": {
        "@name": "what-we-believe",
        "@path": "/travel/about/what-we-believe",
        "hideInNav": "false",
        "title": "What We Believe",
        "title_de": "Woran wir glauben",
        "@nodes": []
    },
    "careers": {
        "@name": "careers",
        "@path": "/travel/about/careers",
        "hideInNav": "false",
        "title": "Careers",
        "title_de": "Karriere",
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

### Querying nodes

```
GET /delivery/{workspace}/v1?param1=value1&param2=value2
```

| Parameter | Default value | Description | Parameter type | Data type |
| ------ | ------ | ------ | ------ | ------ |
| ```workspace``` | | [workspace](https://documentation.magnolia-cms.com/display/DOCS/Workspaces) | path | string |
| ```nodeTypes``` | mgnl:content, mgnl:contentNode | Comma-separated string of [primary node types](https://documentation.magnolia-cms.com/display/DOCS/Node+types#Nodetypes-Magnoliaprimarynodetypes) | query | string |
| ```q``` |  | Text to search (full-text search) | query | string |
| ```orderBy``` | | Properties to order. For example: "mgnl:lastModified desc,title asc" | query | string |
| ```offset``` | 0 |  Starting position in result list | query | integer |
| ```limit``` | 10 | The number of nodes displayed in a page | query | integer |

#### Listing example
Given [the YAML configuration](#yaml-configuration), we can make the following request
```sh
curl -u superuser:superuser "http://<yourDomain>/.rest/delivery/website/v1?nodeTypes=mgnl:page&limit=2"
```

#### Listing response
```json
[
    {
        "@name": "about",
        "@path": "/travel/about",
        "hideInNav": "false",
        "title": "About",
        "title_de": "Über uns",
        "main": {
            "@name": "main",
            "@path": "/travel/about/main",
            "@nodes": []
        },
        "footer": {
            "@name": "footer",
            "@path": "/travel/about/footer",
            "@nodes": []
        },
        "company": {
            "@name": "company",
            "@path": "/travel/about/company",
            "hideInNav": "false",
            "title": "Our Company",
            "title_de": "Unser Unternehmen",
            "@nodes": []
        },
        "what-we-believe": {
            "@name": "what-we-believe",
            "@path": "/travel/about/what-we-believe",
            "hideInNav": "false",
            "title": "What We Believe",
            "title_de": "Woran wir glauben",
            "@nodes": []
        },
        "careers": {
            "@name": "careers",
            "@path": "/travel/about/careers",
            "hideInNav": "false",
            "title": "Careers",
            "title_de": "Karriere",
            "@nodes": []
        },
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
        "title_de": "über diese Demo",
        "main": {
            "@name": "main",
            "@path": "/travel/meta/about-demo/main",
            "@nodes": []
        },
        "footer": {
            "@name": "footer",
            "@path": "/travel/meta/about-demo/footer",
            "@nodes": []
        },
        "@nodes": [
            "main",
            "footer"
        ]
    }
]
````

#### Full-text search example
```sh
curl -u superuser:superuser "http://<yourDomain>/.rest/delivery/stories/v1?nodeTypes=mgnl:block&q=Zermatt&limit=2"
```

#### Full-text search response
```json
[
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
```

####  Filtering example
```sh
curl -u superuser:superuser "http://<yourDomain>/.rest/delivery/website/v1?title=Our+Company"
```

#### Filtering response
```json
[
    {
        "@name": "company",
        "@path": "/travel/about/company",
        "hideInNav": "false",
        "title": "Our Company",
        "title_de": "Unser Unternehmen",
        "main": {
            "@name": "main",
            "@path": "/travel/about/company/main",
            "@nodes": []
        },
        "footer": {
            "@name": "footer",
            "@path": "/travel/about/company/footer",
            "@nodes": []
        },
        "@nodes": [
            "main",
            "footer"
        ]
    }
]
```

#### Filtering special properties
+ ```@name```: Name of node
+ ```@parent```: Path to parent node

## Limitation




