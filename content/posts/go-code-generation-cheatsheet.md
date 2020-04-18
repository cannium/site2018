---
title: Go Code Generation Cheatsheet
date: 2019-07-14
---

Golang is blamed for lacking of generics, and the solution for that is code generation. Surely reflection could also do the job, but code generation has 3 advantages:

- Generated code could be made type-safe, you don't want to see `interface{}` everywhere
- Generated code runs faster, no runtime overhead
- Generated code is easier to understand and less cryptic

Basically the only library needed is [`text/template`](https://golang.org/pkg/text/template/), and skeleton code looks like(error handling omitted):

```go
t := template.New("exampleTemplate")
t, _ = t.Parse(fileHeader)
t, _ = t.Parse(file)
t, _ = t.Parse(someOtherDefinition)
_ = t.ExecuteTemplate(file, "TemplateEntryName", toRender)
```

First create a new template with name `exampleTemplate`, then add template definitions to it. A template definition works like a function, you could call it in another template. For example:

```go
const fileHeader = `
{{define "FileHeader"}}
// This file is generated. DO NOT EDIT
{{end}}
`
const someOtherDefinition = `
{{define "SomeOtherDefinition"}}
{{ .Name }} := {{ .Value }}
{{end}}
`
const file = `
{{define "TemplateEntryName"}}

{{template "FileHeader"}}

{{range .}}
	{{template "SomeOtherDefinition" .}}
{{end}}

{{end}}
`
```

When `t.ExecuteTemplate` is called, the template engine would first evaluate `TemplateEntryName` defined in `file` variable. `TemplateEntryName` calls the other two templates, `FileHeader` and `SomeOtherDefinition`. `FileHeader` prints a header declaring the file is generated, and `SomeOtherDefinition` is more interesting because it accepts a variable. According to the example above, type of `toRender` could be:

```go
type structToRender struct {
    Name  string
    Value string
}
var toRender []structToRender
```

In `TemplateEntryName`, `{{range .}}` would get a variable typed `[]structToRender` as `.`, and `{{template "SomeOtherDefinition" .}}` would get a variable typed `structToRender`. Then inside `SomeOtherDefinition`, `.Name` and `.Value` both have type `string`.

And a few more tips:

- Use [`go/format`](https://golang.org/pkg/go/format/) to format code before writing to a file
- Having the list to render sorted to generate stable output
- A real [cheatsheet](https://curtisvermeeren.github.io/2017/09/14/Golang-Templates-Cheatsheet)