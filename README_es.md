# Diseño estándar de un proyecto en Go

Traducciones:

* [한국어 문서](README_ko.md)
* [中文文档](README_zh.md)
* [Français](README_fr.md)
* [Spanish](README_es.md)

## Resumen

Este es un diseño básico para proyectos desarrollados con Go. No es un estándar oficial definido por el equipo de desarrollo principal de Go; sin embargo, si es un conjunto de patrones de diseño de proyectos emergentes e históricos comunes dentro del ecosistema Go. Algunos de estos patrones son más populares que otros. También tiene una serie de pequeñas mejoras junto con varios directorios de soporte, comunes a cualquier aplicación del mundo real lo suficientemente grande.

Si estás tratando de aprender Go o si estás construyendo un PoC o un proyecto de prueba, este diseño de proyecto es excesivo. Es recomendable comenzar con algo realmente simple (un solo archivo `main.go` es más que suficiente). A medida que tu proyecto crezca, ten en cuenta que será importante asegurarse de que el código esté bien estructurado, de lo contrario, terminarás con código desordenado con muchas dependencias ocultas y con estados globales. Cuando existan más personas trabajando en el proyecto, necesitarás una buena estructura. Ahí es cuando es importante introducir una forma común de administrar paquetes / librerias. Cuando tienes un proyecto open source o cuando sabes que otros proyectos importan el código desde el repositorio de tu proyecto, es cuando realmente importa tener paquetes y código privado (también conocidos como "internals"). ¡Cloná el repositorio, conservá lo que necesites y eliminá todo lo demás! El hecho de que esté allí no significa que tengas que usarlo todo. Ninguno de estos patrones se utiliza en todos los proyectos. Incluso el patrón "vendedor" no es universal.

Con Go 1.14 [`Go Modules`](https://github.com/golang/go/wiki/Modules) está finalmente listo para ser usado en producción. Usa [`Go Modules`](https://blog.golang.org/using-go-modules) a menos que tengas una razón específica para no hacerlo y si lo haces, entonces no necesitas preocuparte por $GOPATH y ni donde pones tu proyecto. El archivo básico `go.mod` asume que tu proyecto está alojado en GitHub, pero esto no es un requisito. La ruta del módulo puede ser cualquier cosa, aunque el primer componente de la ruta del módulo debe tener un punto en su nombre (la versión actual de Go ya no lo aplica, pero si estás usando versiones un poco más antiguas, no te sorprendas si tus compilaciones fallan sin eso). Si quieres saber más al respecto consulta los problemas registrados en [`37554`](https://github.com/golang/go/issues/37554) y [` 32819`](https://github.com/golang/go/issues/32819).

Este diseño de proyecto es intencionalmente genérico y no intenta imponer una estructura específica para un proyecto desarrollado en Go.

Este es un esfuerzo comunitario. Abre un issue si ves un patrón nuevo o si crees que uno de los patrones existentes debe ser actualizado.

Si necesitas ayuda con los nombres, el formato y/o el estilo de codificación, utiliza [`gofmt`] (https://golang.org/cmd/gofmt/) y [` golint`] (https://github.com/golang/lint ). También asegúrete de leer estas recomendaciones y pautas de estilo de código en Go:

* https://talks.golang.org/2014/names.slide
* https://golang.org/doc/effective_go.html#names
* https://blog.golang.org/package-names
* https://github.com/golang/go/wiki/CodeReviewComments
* [Style guideline for Go packages](https://rakyll.org/style-packages) (rakyll/JBD)

Para obtener información adicional consulta [`Go Project Layout`](https://medium.com/golang-learn/go-project-layout-e5213cdcfaa2).

Más información sobre cómo nombrar y organizar paquetes, así como otras recomendaciones de como estructurar tu código:
* [GopherCon EU 2018: Peter Bourgon - Best Practices for Industrial Programming](https://www.youtube.com/watch?v=PTE4VJIdHPg)
* [GopherCon Russia 2018: Ashley McNamara + Brian Ketelsen - Go best practices.](https://www.youtube.com/watch?v=MzTcsI6tn-0)
* [GopherCon 2017: Edward Muller - Go Anti-Patterns](https://www.youtube.com/watch?v=ltqV6pDKZD8)
* [GopherCon 2018: Kat Zien - How Do You Structure Your Go Apps](https://www.youtube.com/watch?v=oL6JBUk6tj0)

Una publicación en idioma chino sobre las pautas de diseño orientado a paquetes y la capa de arquitectura:
* [面向包的设计和架构分层](https://github.com/danceyoung/paper-code/blob/master/package-oriented-design/packageorienteddesign.md)

## Directorios en GO

### `/cmd`

Principales aplicaciones de tu proyecto

El nombre del directorio de cada aplicación debe coincidir con el nombre del ejecutable que deseas tener (e.g., `/cmd/myapp`).

No pongas mucho código en el directorio de la aplicación. Si crees que el código se puede importar y usar en otros proyectos, entonces debería estar en el directorio `/pkg`. Si el código no es reutilizable o si no deseas que otros lo reutilicen, colocalo en el directorio `/internal`. Te sorprenderá lo que harán los demás, ¡así que sé explícito sobre tus intenciones!

Es común tener una función `main` muy pequeña que importa e invoca el código que se encuentra dentro de los directorios` /internal` y `/pkg` y nada más.

Para ver un ejemplo, consulta el directorio [`/cmd`](cmd/README.md).

### `/internal`

Aplicación privada y librerias. Este es el código que deseas que otros no importen en sus aplicaciones o librerias. Ten en cuenta que el propio compilador de Go aplica este mismo patrón de diseño. Para obtener más detalles consulta Go 1.4 [`notas de la versión`](https://golang.org/doc/go1.4#internalpackages). Ten en cuenta que no estás limitado al directorio `internal` del nivel superior. Puedes tener más de un directorio `internal` en cualquier nivel del árbol de carpetas de tu proyecto.

Opcionalmente, puedes agregar alguna estructura adicional de directorios para separar el código que desaas compartir del que no. No es un requerimiento obligatorio (especialmente para proyectos más pequeños), pero es bueno tener una idea visual que muestre el uso deseado de cada paquete. El código de tu aplicación real puede ir dentro del directorio `/internal/app` (por ejemplo,`/interna/app/myapp`) y el código compartido por esas aplicaciones puede estar ubicado en el directorio `/internal/pkg` (ejemplo,`/internal/pkg/myprivlib`).

### `/pkg`

Código de librerias que puede ser usadas por aplicaciones externas (ejemplo, `/pkg/mypubliclib`). Otros proyectos importarán estas librerias esperando que funcionen, así que piénselo dos veces antes de poner algo aquí :-) Ten en cuenta que el directorio `internal` es una mejor manera de garantizar que tus paquetes privados no se puedan importar porque Go lo aplica. El directorio `/pkg` sigue siendo una buena forma de comunicar explícitamente que el código de ese directorio es seguro para que lo utilicen otros. La publicación de blog [`I'll take pkg over internal`](https://travisjeffery.com/b/2019/11/i-ll-take-pkg-over-internal/) Travis Jeffery proporciona una buena descripción general de los directorios `pkg` e` internal` y cuándo podría tener sentido usarlos.

También es una forma de agrupar el código Go en un solo lugar cuando su directorio raíz contiene muchos componentes y directorios que no son de Go, lo que facilita la ejecución de varias herramientas de Go (como se menciona en estas charlas: [`Best Practices for Industrial Programming`](https://www.youtube.com/watch?v=PTE4VJIdHPg) de GopherCon EU 2018, [GopherCon 2018: Kat Zien - How Do You Structure Your Go Apps](https://www.youtube.com/watch?v=oL6JBUk6tj0) y [GoLab 2018 - Massimiliano Pippi - Patrones de diseño del proyecto en Go](https://www.youtube.com/watch?v=3gQa1LWwuzk)).

Consulte el directorio [`/pkg`](pkg/README.md) si desea ver qué repositorios Go populares usan este patrón de diseño. Este es un patrón de diseño común, pero no se acepta universalmente y algunos miembros de la comunidad de Go no lo recomiendan.

Está bien no usarlo si el proyecto de tu aplicación es realmente pequeño y donde un nivel adicional de anidamiento no agrega mucho valor (a menos que realmente quiera :-)). Piénsalo cuando sea lo suficientemente grande y tu directorio raíz esté bastante ocupado (especialmente si tiene muchos componentes de aplicaciones que no son de Go).

### `/vendor`

Dependencias de tu aplicación (administradas manualmente o por tu herramienta de administración de dependencias favorita, como la nueva función incorporada [`Go Modules`](https://github.com/golang/go/wiki/Modules)). El comando `go mod vendor` creará el directorio `/vendor`. Ten en cuenta que es posible que debas agregar la opción `-mod=vendor` al comando `go build` si no estás usando Go 1.14 donde está activado de forma predeterminada.

No hagas el commit de las dependencias de tu aplicación, si está creando una librería.

Ten en cuenta que desde [`1.13`](https://golang.org/doc/go1.13#modules) Go también habilitó la función proxy (usando [`https://proxy.golang.org`](https://proxy.golang.org) como su servidor proxy por defecto). Lea más sobre esto [`aquí`](https://blog.golang.org/module-mirror-launch) para ver si se ajusta a todos sus requisitos y restricciones. Si es así, entonces no necesitarás el directorio `vendor` en absoluto.

## Directorio de Aplicaciones de Servicios

### `/api`

En este directorio se pueden incluir especificaciones de OpenAPI / Swagger, archivos de esquema JSON o archivos de definición de protocolo.

Para ver un ejemplo, consulta el directorio [`/api`](api/README.md).

## Directorio de Aplicaciones Web

### `/web`

Componentes específicos de tu aplicación web: archivos estáticos, templates del lado del servidor y SPAs.

## Directorios de Aplicaciones Comunes

### `/configs`

Templates de archivos de configuración o configuraciones predeterminadas.

Coloca tus archivos de template `confd` o` consul-template` aquí.

### `/init`

Configuraciones del administrador/supervisor del sistema init (systemd, upstart, sysv) y procesos (runit, supervisord).

### `/scripts`

Scripts varios para realizar algunas operaciones como el build, install y analysis de tu aplicación.

Estos scripts te permiten mantener el archivo Makefile pequeño y simple(ejemplo, [`https://github.com/hashicorp/terraform/blob/master/Makefile`](https://github.com/hashicorp/terraform/blob/master/Makefile)).

Para ver un ejemplo, consulta el directorio [`/scripts`](scripts/README.md).

### `/build`

Empaquetado e Integración Continua.

Coloca tus configuraciones de paquetes tus archivos de cloud (AMI), contenedores (Docker), OS (deb, rpm, pkg) y scripts en el directorio `/build/package`.

Coloca tus configuraciones y scripts de CI (travis, circle, drone) en el directorio `/build/ci`. Ten en cuenta que algunas de las herramientas de CI (por ejemplo, Travis CI) son muy exigentes con la ubicación de tus archivos de configuración. Intenta poner los archivos de configuración en el directorio `/build/ci` vinculándolos a la ubicación donde las herramientas de CI los esperan (cuando sea posible).

### `/deployments`

Configuraciones y templates de implementación de IaaS, PaaS, sistema y orquestación de contenedores (docker-compose, kubernetes / helm, mesos, terraform, bosh). Ten en cuenta que en algunos repositorios (especialmente las aplicaciones implementadas con kubernetes) este directorio se llama `/deploy`.

### `/test`

Aplicaciones de prueba externas adicionales y datos de prueba. Siéntase libre de estructurar el directorio `/test` de la forma que desees. Para proyectos más grandes, tiene sentido tener un subdirectorio de datos. Por ejemplo, puedes tener `/test/data` o `/test/testdata` si necesitas Go para ignorar lo que hay en ese directorio. Ten en cuenta que Go también ignorará los directorios o archivos que comiencen con "." o "_", por lo que tienes más flexibilidad en términos de cómo nombrar tu directorio de datos de prueba.

Para ver un ejemplo, consulta el directorio [`/test`](test/README.md).

## Otros Directorios

### `/docs`

Diseño y documentos de usuario (además de tu documentación generada por godoc).

Para ver un ejemplo, consulta el directorio [`/docs`](docs/README.md).

### `/tools`

Herramientas de soporte para este proyecto. Ten en cuenta que estas herramientas pueden importar código de los directorios `/pkg` e` /internal`.

Para ver un ejemplo, consulta el directorio [`/tools`](tools/README.md).

### `/examples`

Ejemplos de tu aplicación y/o librerias publicas.

Para ver un ejemplo, consulta el directorio [`/examples`](examples/README.md).

### `/third_party`

Herramientas externas, código forkeado y otras utilidades de terceros (ejemplo, Swagger UI).

### `/githooks`

Hooks de git.

### `/assets`

Otros archivos estáticos que son utilizados en tu repositorio (imagenes, logos, etc).

### `/website`

Si no estás utilizando las páginas de GitHub, este es el lugar correcto para colocar los datos que corresponden al sitio web de tu proyecto.

Para ver un ejemplo, consulta el directorio [`/website`](website/README.md).

## Directorios que no deberias tener

### `/src`

Algunos proyectos de Go tienen el directorio `src`, para esto usualmente ocurre cuando los desarrolladores provienen del mundo Java, donde generar este tipo de directorio es una práctica común. Realmente no quieres que tu código Go o proyectos Go se parezcan a Java :-)

No confundas el directorio de nivel de proyecto `/src` con el directorio` /src` que Go usa para sus espacios de trabajo como se describe en [`Cómo escribir código de Go`](https://golang.org/doc/code.html ). La variable de entorno `$GOPATH` apunta a tu espacio de trabajo (actual) (de forma predeterminada, apunta a` $HOME/go` en sistemas que no son Windows). Este espacio de trabajo incluye los directorios de nivel superior `/pkg`,` /bin` y `/src`. Tu proyecto real termina siendo un subdirectorio en `/src`, por lo que si tienes en tu proyecto el directorio` /src`, la ruta del proyecto se verá así: `/some/path/to/workspace/src/your_project/src/your_code.go`. Ten en cuenta que con Go 1.11 es posible tener tu proyecto fuera de tu `GOPATH`, pero aún así esto no significa que sea una buena idea usar este patrón de diseño.

## Badges
* [Go Report Card](https://goreportcard.com/) - Escaneará tu código con `gofmt`, `go vet`, `gocyclo`, `golint`, `ineffassign`, `license` y `misspell`. Reemplaza `github.com/golang-standards/project-layout` con la referencia de tu proyecto.

* ~~[GoDoc](http://godoc.org) - Proporciona una versión en linea de tu documentación generada con GoDoc. Cambia el link para que apunte a tu proyecto.~~

* [Pkg.go.dev](https://pkg.go.dev) - Pkg.go.dev is a new destination for Go discovery & docs. You can create a badge using the [badge generation tool](https://pkg.go.dev/badge).

* Release - Mostrará el número de versión más reciente de tu proyecto. Cambia el link para que apunte a tu proyecto.

[![Go Report Card](https://goreportcard.com/badge/github.com/golang-standards/project-layout?style=flat-square)](https://goreportcard.com/report/github.com/golang-standards/project-layout)
[![Go Doc](https://img.shields.io/badge/godoc-reference-blue.svg?style=flat-square)](http://godoc.org/github.com/golang-standards/project-layout)
[![PkgGoDev](https://pkg.go.dev/badge/github.com/golang-standards/project-layout)](https://pkg.go.dev/github.com/golang-standards/project-layout)
[![Release](https://img.shields.io/github/release/golang-standards/project-layout.svg?style=flat-square)](https://github.com/golang-standards/project-layout/releases/latest)

## Notas

Un template de proyecto más obstinado con configuraciones, scripts y código de muestra/reutilizables es un WIP.
