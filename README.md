# Blocks

Blocks is a, simple, Go-idiomatic view engine based on [html/template](https://pkg.go.dev/html/template?tab=doc#Template), plus the following features:

- Compatible with the [http.FileSystem](https://golang.org/pkg/net/http/#FileSystem) interface
- Embedded templates through [go-bindata](https://github.com/go-bindata/go-bindata)
- Load with optional context for cancelation
- Reload templates on development stage
- Full Layouts and Blocks support
- Markdown Content
- Global [FuncMap](https://pkg.go.dev/html/template?tab=doc#FuncMap)

## IMPORTANT: The original author is [@kataras](https://github.com/kataras), this repository is a modified version of [Blocks](https://github.com/kataras/blocks/) containing minor fixes and corrections. 

## Installation

The only requirement is the [Go Programming Language](https://golang.org/dl).

The original repository created by kataras can be installed using the following command.

```sh
$ go get github.com/kataras/blocks
```

**If you want to use this repository containing fixes to the original project you must download it and add it to your project directly.**

## Getting Started

Create a folder named **./views** and put some HTML template files.

```
│   main.go
└───views
    |   index.html
    ├───layouts
    │       main.html
    ├───partials
    │       footer.html
```

Now, open the **./views/layouts/main.html** file and paste the following:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ if .Title }}{{ .Title }}{{ else }}Default Main Title{{ end }}</title>
</head>
<body>
    {{ template "content" . }}

<footer>{{ partial "partials/footer" .}}</footer>
</body>
</html>
```

The `template "content" .` is a Blocks builtin template function which renders the **main** content of your page that should be rendered under this layout. The block name should always be `"content"`.

The `partial` is a Blocks builtin template function which renders a template inside other template.

The `.` (dot) passes the template's root binding data to the included templates.

Continue by creating the **./views/index.html** file, copy-paste the following markup:

```html
<h1>Index Body</h1>
```

In order to render `index` on the `main` layout, create the `./main.go` file by following the below.

Import the package:

```go
import "github.com/kataras/blocks"
```

The `blocks` package is fully compatible with the standard library. Use the [New(directory string)](https://pkg.go.dev/github.com/kataras/blocks?tab=doc#New) function to return a fresh Blcoks view engine that renders templates. 

This directory can be used to locate system template files or to select the wanted template files across a range of embedded data (or empty if templates are not prefixed with a root directory).

```go
views := blocks.New("./views")
```

The default layouts directory is `$dir/layouts`, you can change it by `blocks.New(...).LayoutDir("otherLayouts")`.

To parse files that are translated as Go code, inside the executable program itself, pass the [go-bindata's generated](https://github.com/go-bindata/go-bindata) latest version's `AssetFile` method to the `New` function:

```sh
$ go get -u github.com/go-bindata/go-bindata
```

```go
views := blocks.New(AssetFile())
```

After the initialization and engine's customizations the user SHOULD call its [Load() error](https://pkg.go.dev/github.com/kataras/blocks?tab=doc#Blocks.Load) or [LoadWithContext(context.Context) error](https://pkg.go.dev/github.com/kataras/blocks?tab=doc#Blocks.LoadWithContext) method once in order to parse the files into templates.

```go
err := views.Load()
```

To render a template through a compatible [io.Writer](https://golang.org/pkg/io/#Writer) use the [ExecuteTemplate(w io.Writer, tmplName, layoutName string, data interface{})](https://pkg.go.dev/github.com/kataras/blocks?tab=doc#Blocks.ExecuteTemplate) method.

```go
func handler(w http.ResponseWriter, r *http.Request) {
	data := map[string]interface{}{
		"Title": "Index Title",
	}

	err := views.ExecuteTemplate(w, "index", "main", data)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
}
```

There are several methods to customize the engine, **before `Load`**, including `Delims`, `Option`, `Funcs`, `Extension`, `RootDir`, `LayoutDir`, `LayoutFuncs`, `DefaultLayout` and `Extensions`. You can learn more about those in our [godocs](https://pkg.go.dev/github.com/kataras/blocks?tab=Blocks).

Please navigate through [_examples](_examples) directory for more.

## License

This software is licensed under the [MIT License](LICENSE).
