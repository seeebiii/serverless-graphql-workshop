# Lab 6 - Pagination

## In this lab …

* Extends comments field to handle arguments
* Extend DynamoDB request to handle pagination

## Implementation

1. Go back to the [AppSync console](console.aws.amazon.com/appsync) and select the API
2. In the sidebar, click on **Schema**
3. Introduce the new type `CommentConnection` and extend the `Article` type to support the `limit` and `nextToken` arguments for comments:
    ```graphql
    type Article {
      id: ID!
      createdAt: AWSDateTime!
      title: String!
      content: String!
      comments(limit: Int, nextToken: String): CommentConnection!
    }

    type CommentConnection {
      nodes: [Comment!]!
      nextToken: String
    }
    ``` 
4. Click on **Save schema**
5. Find **comments(...): CommentConnection!** in the list of resolvers and click on **comments**
6. Replace the request mapping template:
    ```velocity
    {
        "version" : "2017-02-28",
        "operation" : "Query",
        "query": {
            "expression": "articleId = :articleId",
            "expressionValues" : {
                ":articleId" : $util.dynamodb.toDynamoDBJson($ctx.source.id)
            }
        }
        #if($ctx.args.limit)
            ,"limit": $util.toJson($ctx.args.limit)
        #end
        #if($ctx.args.nextToken)
            ,"nextToken": $util.toJson($ctx.args.nextToken)
        #end
    }
    ```
7. And the response mapping template:
    ```velocity
    {
        "nodes": $utils.toJson($ctx.result.items)
        #if($ctx.result.nextToken)
            ,"nextToken": $util.toJson($ctx.result.nextToken)
        #end
    }
    ```
8. Click on **Save resolver**
9. Go to the **Queries** tool and run this query:
    ```graphql
    query {
      article(id: "<< YOUR ARTICLE ID >>") {
        id
        title
        comments(limit: 2) {
          nodes {
            id
            createdAt
            content
          }
          nextToken
        }
      }
    }
    ```

If you have more than two comments, then you should get back a `nextToken` as part of the response. You can use the `nextToken` to query the next chunk of comments. This is the easiest solution we can implement to support pagination.

## Questions

* Is it recommended to implement pagination from the beginning?