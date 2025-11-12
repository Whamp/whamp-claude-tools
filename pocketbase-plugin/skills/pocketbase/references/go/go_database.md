# Database Operations - Go Extensions

## Overview

PocketBase provides a powerful database API for Go extensions, allowing you to perform complex queries, transactions, and data operations.

## Basic Queries

### Find Records

```go
import "github.com/pocketbase/dbx"

// Find single record
record, err := app.FindRecordById("posts", "RECORD_ID")

// Find multiple records with filter
records, err := app.FindRecordsByFilter(
    "posts",
    "status = {:status}",
    "-created",
    50,
    0,
    dbx.Params{"status": "published"},
)

// Fetch all matching records without pagination
records, err = app.FindRecordsByFilter(
    "posts",
    "status = {:status}",
    "-created",
    0,
    0,
    dbx.Params{"status": "published"},
)
```

### Query Builder

```go
import (
    "github.com/pocketbase/dbx"
    "github.com/pocketbase/pocketbase/core"
)

records := []*core.Record{}
err := app.RecordQuery("posts").
    AndWhere(dbx.HashExp{"status": "published"}).
    AndWhere(dbx.NewExp("created >= {:date}", dbx.Params{"date": "2024-01-01"})).
    AndWhere(dbx.Or(
        dbx.HashExp{"author": userId},
        dbx.HashExp{"featured": true},
    )).
    OrderBy("created DESC").
    Offset(0).
    Limit(50).
    All(&records)
```

### Transactions

```go
import "github.com/pocketbase/pocketbase/core"

err := app.RunInTransaction(func(txApp core.App) error {
    collection, err := txApp.FindCollectionByNameOrId("posts")
    if err != nil {
        return err
    }

    post := core.NewRecord(collection)
    post.Set("title", "New Post")
    if err := txApp.Save(post); err != nil {
        return err
    }

    commentsCol, err := txApp.FindCollectionByNameOrId("comments")
    if err != nil {
        return err
    }

    comment := core.NewRecord(commentsCol)
    comment.Set("post", post.Id)
    comment.Set("content", "First comment")
    return txApp.Save(comment)
})
```

---

**Note:** See [go_overview.md](go_overview.md) and the [official database guide](https://pocketbase.io/docs/go-database/) for comprehensive coverage.
