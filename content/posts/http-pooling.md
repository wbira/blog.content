---
title: "Http Pooling"
date: 2020-03-26T21:17:49+01:00
description: "Lambda provisioned concurrency"
tags: [
  "angular",
  "http",
  "pooling"
]
type: "post"
---

### HTTP polling in RxJs

Hello! Today post will be quick one. I'll show you little utility, that I created for testing HTTP pooled based API.
Testing such API's via curl or Postman can by really annoying. I had some frontend background, so I thought that RxJS can solve my problem in a few lines, in nice declarative way.

```javascript
import { catchError, map, pluck, tap, mergeMap, take, delay, filter, scan } from 'rxjs/operators';


const ERROR_MSG = "Could not fetch data. Please try again later"
const maxAttempts = 10


function checkAttempts(maxAttempts: number) {
  return (attempts: number) => {
    if (attempts > maxAttempts) {
      throw new Error("Error: max attempts");
    }
  };
}

export function httpPoll(
  httpFetcher: (number) => Observable<string>,
  predicate: (string) => boolean): Observable<string> {
  return timer(0, 1000).pipe(
      scan((acc) => ++acc, 0),
      tap(checkAttempts(maxAttempts)),
      mergeMap((attempt) => httpFetcher(attempt)),
      filter(status => predicate(status)),
      take(1),
      catchError((err) => of(ERROR_MSG))
    )
}

```

Additionaly for this blog post, I created [example on stackblitz](https://stackblitz.com/edit/angular-ez6ygg?embed=1&file=src/app/polling.ts), so you could check how it works.



