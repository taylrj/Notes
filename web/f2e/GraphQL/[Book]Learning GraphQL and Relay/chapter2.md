# Chapter2. The Query Language

##  Explore the language syntax
### Example: A query to read the list of comments on a blog article

```grapghql
  query ArticleComments($articleId: Int!) {
    article(articleId: $articleId){
      comments {
        commentId
        formattedBody
        timestamp
      }
    }
  }
```
- The type of operation is a query, and the name is  `ArticleComments`

- The string start with '$' stands for a variable, and the type of the variable is `Int`. Note that the trailing `!` means the variable is required and can not be null

- To execute the generic query, we supply a JSON object for the variables input like: 

```json
  "articleId" : 42
```

- This is a possible response:

```graphql
  {
    "article":{
      "comments": [
        {
          "commentId": 1,
          "formattedBody": "GraphQL is <strong>cool</strong>",
          "timestamp": "12/12/2015 - 12:12"
        },
        {
          ...
        }
      ]
    }
  }
```
- Therefore, we can think of fields as functions cause they return something in the response
- The curly braces in the GraphQL query are called selection sets, and those are nestable, which means one selection set can contain other selection sets

### Directives
We can provide options to alter the GraphQL runtime execution using directives.
- `@include`, which directs the GraphQL executor to include a field or a fragment only when the `if` argument is true
```graphql
field @include(if: $BooleanValue)
```
- `@skip`

Those two main build-in directives that should be supported by every GraphQL server

### Fragments
Using the spread operator
```graphql
query TwoArticles {
  firstArticle: article(articleId: 42) {
    ...CommentList
  }
  secondArticle: article(articleId: 43) {
    ...CommentList
  }
}

fragment CommentList on Article {
  comments {
    commentId
    formattedBody
    timestamp
  }
}
```

### Mutations
Most clients will communicate their need to update the data.
```graphql
mutation AddNewComment {
  addComment(
    articleId: 42,
    authorEmail: "mark@gmail.com",
    mardkown: "Hello world"
  ){
    id,
    formattedBody
    timestamp
  }
}
```
- The `addComment` function on the server will persist the comment data that is received through arguments, and return a JSON object to UI, which includes id, formattedBoy...