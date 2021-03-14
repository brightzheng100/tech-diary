# Go Template Walkthrough Guide

## Concepts & Syntax

They’re delimited by `{{ }}`. Other, other non-delimited parts are left untouched.

### Data evaluations

To obtain data from a struct, you can use the `{{ .FieldName}}` action, which will replace it with the `FieldName` value of the given struct, on parse time. The struct is given to the `Execute` function, which we’ll cover later.

There’s also the `{{.}}` action that you can use to refer to a value of non-struct types.

### Conditions

`if`,`else if`,`end` structure is supported. For example:

```text
{{if .FieldName}} Value of FieldName is {{ .FieldName }} {{end}}

Or

{{if .FieldName}} action {{ else }} action 2 {{end}}
```

### Loops

Using the `range` action you can loop through a slice. A `range` actions is defined using the `{{range .Member}} ... {{end}}` template.

If your slice is a non-struct type, you can refer to the value using the `{{ . }}` action. In case of structs, you can refer to the value using the `{{ .Member }}` action, as already explained.

### Functions, Pipelines and Variables

Actions have several built-in functions that are used along with pipelines to additionally parse output. Pipelines are annotated with `|` and the default behavior is sending data from the left side to the function to the right side.

Functions are used to escape the action’s result. There’re several functions available by default such as, `html` which returns HTML escaped output, safe against code injection or `js` which returns JavaScript escaped output.

Using the `with` action, you can define variables that are available in that `with` block: `{{ with $x := <^>result-of-some-action<^> }} {{ $x }} {{ end }}`.

## Parsing Templates

The three most important and most frequently used functions are:

* `New` — allocates new, undefined template,
* `Parse` — parses given template string and return parsed template,
* `Execute` — applies parsed template to the data structure and writes result to the given writer.

Let's run through a sample:

```text
$ mkdir go-template && cd go-template

$ cat > main.go <<EOF
package main

import (
	"os"
	"text/template"
)

type Todo struct {
	Name        string
	Description string
}

func main() {
	td := Todo{"Test templates", "Let's test a template to see the magic."}

	t, err := template.New("todos").Parse("You have a task named \"{{ .Name}}\" with description: \"{{ .Description}}\"")
	if err != nil {
		panic(err)
	}
	err = t.Execute(os.Stdout, td)
	if err != nil {
		panic(err)
	}
}
EOF

$ go run main.go
You have a task named "Test templates" with description: "Let's test a template to see the magic."
```

We can reuse the same template `t`, without needing to create or parse it again by providing the struct you want to use to the `Execute` function again:

```text
  ...
	tdNew := Todo{"Go", "Contribute to any Go project"}
	err = t.Execute(os.Stdout, tdNew)
	if err != nil {
		panic(err)
	}
}
```

As you can see, templates provide a powerful way to customize textual output. Beside manipulating textual output, you can also manipulate HTML output using the `html/template` package.

## Verifying Templates

`template` packages provide the `Must` functions, used to verify that a template is valid during parsing. The `Must` function provides the same result as if we manually checked for the error, like in the previous example.

This approach saves you typing, but if you encounter an error, your application will panic. For advanced error handling, it’s easier to use above solution instead of `Must` function.

The `Must` function takes a template and error as arguments. It’s common to provide `New` function as an argument to it:

```text
t := template.Must(template.New("todos").Parse("You have task named \"{{ .Name}}\" with description: \"{{ .Description}}\""))
```

## Implementing Templates

HTML templating is also common.

```text
$ cat > todo.tpl <<EOF
<!DOCTYPE html>
<html>
  <head>
    <title>Go To-Do list</title>
  </head>
  <body>
    <p>
      To-Do list for user: {{ .User }} 
    </p>
    <table>
      	<tr>
          <td>Task</td>
          <td>Done</td>
    	</tr>
      	{{ with .List }}
			{{ range . }}
      			<tr>
              		<td>{{ .Name }}</td>
              		<td>{{ if .Done }}Yes{{ else }}No{{ end }}</td>
      			</tr>
			{{ end }} 
      	{{ end }}
    </table>
  </body>
</html>
EOF

$ cat > html.go <<EOF
package main

import (
	"html/template"
	"os"
)

type entry struct {
	Name string
	Done bool
}

type ToDo struct {
	User string
	List []entry
}

func main() {
	todos := ToDo{
		User: "Tom",
		List: []entry{
			{
				Name: "TOTO Item 1",
				Done: true,
			},
			{
				Name: "TOTO Item 2",
				Done: false,
			},
		},
	}

	// Files are provided as a slice of strings.
	paths := []string{
		"todo.tpl",
	}

	t, err := template.New("todo.tpl").ParseFiles(paths...)
	if err != nil {
		panic(err)
	}
	err = t.Execute(os.Stdout, todos)
	if err != nil {
		panic(err)
	}
}
EOF

$ go run html.go
```

> Note: In line 67 above, if we use an ad-hoc template name say `html-template` , we would get error: `panic: template: "html-template" is an incomplete or empty template`
>
> ```text
> t, err := template.New("html-template").ParseFiles(paths...)
> ```



## References

{% embed url="https://blog.gopheracademy.com/advent-2017/using-go-templates/" %}

{% embed url="https://www.cnblogs.com/52php/p/6412554.html" %}



