---
layout: post
title:  Serialization in Go
date:   2020-06-05 00:00:00 -0700
categories:
tags:
---

Recently I had a bit of fun and learning in golang serialzation, expecially on
["endcoding/gob"](https://golang.org/pkg/encoding/gob/) and 
["encoding/json"](https://golang.org/pkg/encoding/json/). I thought of 
noting it down.


# On nested types and private fields

Both "encoding/json" and "encoding/gob" deal with nested types, but do not 
serialize private fields by default.

<script src="https://gist.github.com/liusy182/c47c4f6eb5e5096b33813ddeb7367147.js"></script>

# On interface{}

Unfortunately "encoding/gob" does not serialize `interface{}` very well without
proper [type registration](https://golang.org/pkg/encoding/gob/#Register).
Surprisingly though, "encoding/json" is able to do it well. I guess the reason is
because `gob` stores type information as part of the serialziation result, whereas
`json` does not.

<script src="https://gist.github.com/liusy182/4070b0d6474913dee8f0b416c3748bd3.js"></script>

# On nil v.s. empty

`gob` does not distinguish between a `nil` pointer and a pointer with no content 
(e.g. pointer to an empty string or an empty slice). This results to very undesirable 
behavior as the original information in a struct is lost after a round of 
serialization and deserialzation. This behavior turns out to be 
[a bug in Go library](https://github.com/golang/go/issues/4609) but was 
closed as working as intended.

The same problem does not occur in `json` library.

<script src="https://gist.github.com/liusy182/7367a8b0a5c8a8efe052a417c3d5a892.js"></script>
