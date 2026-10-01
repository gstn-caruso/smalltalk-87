# Verificación de la revisión

## Expectativa y línea base

Expectativa refutable: la revisión debe distinguir propuesta, resultado observado y límite; citar fuentes primarias próximas a sus afirmaciones; incluir consultas reproducibles; conservar el documento recibido.

Línea base: el repositorio en `f365538` contiene `LICENSE` y `Smalltalk-1987-syntax.md`, sin contraste de 2026. La rama `docs/review-smalltalk-1987-2026` comenzó con árbol limpio. SHA-256 del documento original: `d7b15effadb28bc3f32c44b68644d2bd10aa89e2dbacec1e28d82c6c22026612`.

## Consultas de imagen

Cuis 7.9, actualización 8239; resultados recibidos de la ejecución MCP del coordinador. Evaluar:

```smalltalk
#('1e3' '.5' '16r1e3' '16r1E3' '16r10.0' '1.0e3')
    collect: [:source |
        {source. Compiler new evaluate: source in: nil to: nil
            notifying: nil ifFail: ['PARSE-FAIL']
            logged: false profiled: false}]
```

Resultados, en orden: `1000`, `5`, `4096`, `483`, `16.0`, `1000.0`. Repetir en otra imagen puede producir resultados distintos.

Las otras observaciones del informe proceden de evaluación de bloques, arrays y `thisContext`, más lectura de `BlockClosure>>value`, `MethodContext>>cannotReturn:`, `MethodContext>>closure` y `ContextPart>>return:`. No se instalaron métodos, clases o globales; la evaluación puede actualizar caches u otro estado incidental.

Usando el mismo evaluador, reproducir:

| Expresión | Resultado observado |
|---|---|
| `[:x | x] value: 4` | `4` |
| `[:x || t | t := x. t] value: 4` | `4` |
| `[ | t | t := 2. t] value` | `2` |
| `| action | action := [42]. true ifTrue: action` | `42` |
| `true ifTrue: [thisContext]` | `UndefinedObject>>DoIt` |
| `true ifTrue: [thisContext closure]` | `nil` |
| `| action | action := [thisContext closure]. true ifTrue: action` | Una `BlockClosure` |
| `#(nil true false #foo 'bar' $a)` | El mismo literal |
| `#(foo)` | `#(#foo)` |
| `#(foo (bar))` | `#(#foo #(#bar))` |
| `#'a b'` | El mismo símbolo |
| `{1. 2 + 3}` | `#(1 5)` |
| `| x | x _ 3. x` | `3` |

## Consulta de VMMaker en el puerto 1470

El investigador recuperó JSON-RPC mediante `nc -w5 localhost 1470`, con `tools/list` y `tools/call`; usó las herramientas `eval` y `read-method-source`, sin instalar métodos o clases ni escribir servicios o scripts. Consulta de identidad:

```smalltalk
{Smalltalk version. VMMaker versionString.
 StackInterpreter defaultObjectMemoryClass name}
```

Resultado: `#('Squeak6.2alpha' '4.7.0 (Cog)' #NewObjectMemory)`. El coordinador corroboró identidad mediante `StackInterpreter objectMemoryClass` y leyó `internalCannotReturn:` y `wordIndexableFormat`. `VMMaker class>>versionString` devuelve un literal: no se identificó una revisión exacta del paquete instalado.

Para repetir la lectura con el MCP nativo activo, usar el comando nativo (la respuesta JSON contiene las fuentes):

```sh
nc -w 5 localhost 1470 <<'EOF'
{"jsonrpc":"2.0","id":4,"method":"tools/call","params":{"name":"read-method-source","arguments":{"methods":["StackInterpreter>>internalCannotReturn:","Spur64BitMemoryManager>>wordIndexableFormat","VMMaker class>>versionString"]}}}
EOF
```

Para repetir identidad por ese mismo canal, usar `name: "eval"` con `arguments: {"code": "{Smalltalk version. VMMaker versionString. StackInterpreter objectMemoryClass}"}` en el objeto `params` de la solicitud.

Fuentes leídas: `InterpreterPrimitives>>primitiveClosureValue`, `primitiveFullClosureValue`; `StackInterpreter>>activateNewClosure:outer:method:numArgs:mayContextSwitch:`, `internalCannotReturn:`, `internalAboutToReturn:through:`, `commonCallerReturn`; métodos de formatos de `SpurMemoryManager` y `Spur64BitMemoryManager`. El informe identifica las conclusiones y sus límites; leer clases Spur no verifica que la VM ejecutora use Spur.

## Límites

No se dispuso de las cláusulas del estándar ANSI. No se ejecutó una batería exhaustiva de gramática, redefinición de mensajes optimizados, retornos escapados o concurrencia. No se verificaron identificadores Unicode. La inspección del intérprete no prueba equivalencia de todas las rutas del JIT. Las inconsistencias del archivo no se atribuyen a la edición histórica sin cotejo con facsímil.

## Comprobación documental final

Comprobación realizada: se cotejaron tablas y consultas con los resultados recibidos; se incorporó evidencia de fuente viva de VMMaker y su límite de configuración. Las citas de ANSI, Cuis, Tonel y OpenSmalltalk apuntan a fuentes primarias entregadas por los investigadores. Los tres enlaces locales corresponden a archivos presentes. `git diff --check` no informó errores y el SHA-256 original permaneció idéntico. La revisión del contenido no encontró secretos; el commit se realiza manualmente tras esos checks. No se añadieron tests automatizados porque el cambio es documental: las consultas y el cotejo manual refutan sus afirmaciones concretas.

No se aplica un criterio de refactor de código de producción: no se cambió código. El informe sí respeta la separación de responsabilidades entre parser, imagen y VM. El repositorio no contiene workflows de CI; los checks locales no se presentan como CI verde ni autorizan un merge.
