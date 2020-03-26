---
title: "Http Polling"
date: 2020-03-26T21:17:49+01:00
description: "How to implement client-side http polling with RxJs"
tags: [
  "RxJs",
  "http",
  "polling",
  "javascript"
]
type: "post"
---


Hello! Today post will be a quick one. I'll show you small utility, that I created for testing HTTP long-polling based API.
 <!--more-->



Testing such APIs via curl or Postman can by really annoying. I have some frontend background, so I thought that RxJs can solve my problem in few lines, in nice declarative way.




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



