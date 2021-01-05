# Description

Duplicated IDs in user data, even on different entities (i.e., if author.id=='1' and book.id=='1'), cause data to be lost by gatsby-transformer-json due to gatsby src's [createNode](https://github.com/gatsbyjs/gatsby/blob/121ccbf5feeeffa0a6f87a20fcfc8fd68a71c425/packages/gatsby/src/redux/actions/public.js#L618) requiring globally unique IDs.

On [line 63](https://github.com/gatsbyjs/gatsby/blob/121ccbf5feeeffa0a6f87a20fcfc8fd68a71c425/packages/gatsby-transformer-json/src/gatsby-node.js#L63) and [line 70](https://github.com/gatsbyjs/gatsby/blob/121ccbf5feeeffa0a6f87a20fcfc8fd68a71c425/packages/gatsby-transformer-json/src/gatsby-node.js#L70) of `gatsby-transformer-json/src/gatsby-node.js`, the `id`s that will eventually be passed to [createNode](https://github.com/gatsbyjs/gatsby/blob/121ccbf5feeeffa0a6f87a20fcfc8fd68a71c425/packages/gatsby/src/redux/actions/public.js#L618) are not checked for uniqueness.

Example data: `/data/books.json` and `/data/authors.json`.

```
// books.json
[
  {
    "id": "a"
  },
  {
    "id": "b"
  },
  {
    "id": "c"
  }
]

// authors.json
[
  {
    "id": "a" // will be lost due to ID collision books.a
  },
  {
    "id": "b" // will be lost due to ID collision with books.b
  },
  {
    "id": "d"
  }
]
```

# Instructions to reproduce

1. Clone the repo

2. yarn install

3. yarn start

4. visit `localhost:8000/___graphql` and query

```
query MyQuery {
  allBooksJson {
    edges {
      node {
        id
      }
    }
  }
  allAuthorsJson {
    edges {
      node {
        id
      }
    }
  }
}

```

### Expected Output:

```
{
  "data": {
    "allBooksJson": {
      "edges": [
        {
          "node": {
            "id": "a"
          }
        },
        {
          "node": {
            "id": "b"
          }
        },
        {
          "node": {
            "id": "c"
          }
        }
      ]
    },
    "allAuthorsJson": {
      "edges": [
        {
          "node": {
            "id": "a"
          }
        },
        {
          "node": {
            "id": "b"
          }
        },
        {
          "node": {
            "id": "d"
          }
        }
      ]
    }
  },
  "extensions": {}
}
```

### Actual Output:

```
{
  "data": {
    "allBooksJson": {
      "edges": [
        {
          "node": {
            "id": "a"
          }
        },
        {
          "node": {
            "id": "b"
          }
        },
        {
          "node": {
            "id": "c"
          }
        }
      ]
    },
    "allAuthorsJson": {
      "edges": [
        {
          "node": {
            "id": "d"
          }
        }
      ]
    }
  },
  "extensions": {}
}
```
