# Smalltalk 1987 frente a 2026

La propuesta de julio de 1987 sigue siendo útil para entender decisiones de Smalltalk, pero **no describe por sí sola el lenguaje de una imagen actual**. En Cuis, las clausuras, las temporarias de bloque y el uso de una variable como argumento de un mensaje de control coinciden con parte de su intención. En cambio, los números y la visibilidad de `thisContext` muestran diferencias concretas. La VM contemporánea también mantiene decisiones propias sobre retornos, primitivas y formatos de objetos. Esta revisión conserva el documento histórico y separa lo propuesto, lo observado y lo que todavía requiere pruebas.

Fecha de revisión: **30 de septiembre de 2026**. Documento de partida: [Proposal for 1987 Smalltalk Standard Syntax](../Smalltalk-1987-syntax.md).

## Una propuesta de trabajo no equivale al estándar posterior

El texto se presenta como un *working document*: define sintaxis y trata la semántica sólo parcial e informalmente. Su historial identifica la **versión 6, del 1 de julio de 1987**, enviada a los participantes del taller para aprobación. También reconoce que su conjunto mínimo de clases y mensajes no alcanza como tratamiento de la biblioteca. Esas limitaciones forman parte del documento, no son objeciones agregadas retrospectivamente.

Por eso conviene distinguir tres preguntas: qué quería cambiar frente al Blue Book, qué terminó especificando una norma posterior y qué implementa hoy un dialecto particular. Una coincidencia con Cuis no prueba adopción universal; una diferencia tampoco invalida automáticamente toda la propuesta.

ANSI publicó después **INCITS 319-1998, Information Technology — Programming Languages — Smalltalk**. La ficha consultada permite identificar esa norma; no se dispuso de su texto completo para contrastar cláusulas. En este informe no se atribuye conformidad ANSI a Cuis ni a una VM a partir de esa ficha. ([Ficha ANSI](https://webstore.ansi.org/standards/incits/ansiincits3191998r2007)).

Las comprobaciones de imagen usan **Cuis 7.9, actualización 8239**. El MCP recuperado en el puerto **1470** permitió consultar otra imagen: **Squeak 6.2alpha, VMMaker 4.7.0 (Cog)**. El número de puerto no identifica una versión de lenguaje o de VM. La evidencia ejecutada vale para la imagen consultada; el código de VM inspeccionado vale para la revisión indicada en su enlace.

## Los números conservan diferencias que cambian resultados

La propuesta quería eliminar exponentes en enteros y números fraccionarios en otras bases, aceptar letras minúsculas como dígitos y dar a `.5` el significado de un flotante. La tabla permite comprobar que **Cuis conserva varios comportamientos que el documento quería reemplazar**.

| Fuente evaluada | Resultado anunciado en 1987 | Resultado observado en Cuis |
|---|---|---|
| `1e3` | Ilegal: el exponente exige un punto | `1000` |
| `1.0e3` | `1000.0` | `1000.0` |
| `16r1e3` | `483` | `4096` |
| `16r1E3` | `483` | `483` |
| `16r10.0` | Ilegal: sin flotantes en otra base | `16.0` |
| `.5` | `0.5` | `5` |

El contraste entre `16r1e3` y `16r1E3` importa: cambiar la caja de una letra cambia el valor observado. Tampoco corresponde transportar la notación `.5` desde otro lenguaje sin probarla. Son decisiones del lector numérico y del compilador; inspeccionar el JIT no permite deducir cómo se tokeniza el texto fuente.

La comprobación se ejecutó con cada cadena como un programa completo, usando `Compiler>>evaluate:in:to:notifying:ifFail:logged:profiled:`. El registro de [verificación](../research_notes/Smalltalk%201987%20frente%20a%202026/verificacion.md) conserva la consulta reproducible. No se probaron exhaustivamente bases, exponentes extremos, signos o errores de desbordamiento.

## Las clausuras avanzaron, pero observar contextos sigue siendo delicado

El aporte semántico central del documento consiste en evitar que los argumentos de bloque se guarden en el contexto de su método de origen. Propone clausuras y una activación por invocación, para permitir recursión y usos concurrentes sin el antiguo problema de un bloque ya activo. También agrega temporarias de bloque.

En Cuis se verificaron bloques con argumentos y temporarias. Su clase `BlockClosure` y el método `value`, con `<primitive: 201>`, muestran el mecanismo que usa esa imagen. Esto apoya la comparación con la intención de 1987, pero no constituye una prueba completa de recursión, concurrencia o independencia de todas las activaciones.

La propuesta también exige aceptar un argumento de control que no sea un bloque escrito directamente en el envío. La comprobación con una variable `action` que contiene `[42]`, seguida de `true ifTrue: action`, devolvió **`42`**. Es una coincidencia concreta con esa exigencia; no demuestra por sí sola que cualquier redefinición de los mensajes optimizados preserve todas las propiedades de un envío ordinario.

El resultado impreso de `true ifTrue: [thisContext]` fue `UndefinedObject>>DoIt`, pero ese nombre por sí solo no permite decidir si se trata de una activación de bloque. La consulta adicional de `true ifTrue: [thisContext closure]` devolvió **`nil`**; al ejecutar `| action | action := [thisContext closure]. true ifTrue: action`, devolvió una **`BlockClosure`**. Se leyó también `MethodContext>>closure`, que devuelve `closureOrNil`. Esta asimetría concreta entre bloque en línea y argumento indirecto entra en tensión con la exigencia de representación fiel de `thisContext` en 1987. No se extendió la conclusión a todas las formas de condicional, cadenas de emisores, niveles de optimización o compiladores.

Hay que distinguir el bloque escrito, la clausura que se crea y el contexto que se materializa. La presencia de una clase o una primitiva no basta para probar qué objetos genera cada optimización. Una comprobación de esa propiedad necesita resultados visibles o una traza de ejecución adecuada.

## Retornos, primitivas y objetos tienen contratos de implementación

Para retornos anómalos, el documento propone enviar `resumeWith:` al objeto de destino. Para `^` dentro de un bloque propone enviar `remoteReturn:` al contexto actual. Es una decisión de protocolo observable: no equivale simplemente a tener clausuras.

La imagen Cuis consultada conserva un camino `MethodContext>>cannotReturn:`. Cuando tiene clausura, delega en `cannotReturn:to:`; en el otro camino informa que la computación terminó. `ContextPart>>return:` usa `unwindAndResume:evaluating:`. Estos métodos no son una reproducción literal de los protocolos propuestos.

En el intérprete Spur de 64 bits inspeccionado también aparece el camino de retorno imposible con `cannotReturn:`. El código permite contrastar ese mecanismo con 1987, pero **no prueba que todos los retornos del JIT recorran exactamente ese camino**, ni reemplaza una ejecución de un retorno no local escapado. ([Código de OpenSmalltalk VM, revisión fijada](https://github.com/OpenSmalltalk/opensmalltalk-vm/blob/3ea6bd17fabb5aca6fcbd175185d2fccdd50d919/src/spur64.stack/interp.c)).

La consulta de fuente viva en VMMaker corroboró un camino equivalente: `StackInterpreter>>internalCannotReturn:` prepara un envío normal a `SelectorCannotReturn`. `internalAboutToReturn:through:` usa `SelectorAboutToReturn`. Son métodos leídos en la imagen de desarrollo; no se ejecutaron retornos anómalos para comprobar esos caminos. OpenSmalltalk distingue familias de intérpretes y generadores: identificar Cog no alcanza para demostrar qué configuración concreta está ejecutando la imagen. ([README de OpenSmalltalk VM](https://github.com/OpenSmalltalk/opensmalltalk-vm/blob/Cog/README.md)).

En la misma imagen, `InterpreterPrimitives>>primitiveClosureValue` valida el contexto externo y el método, y activa una nueva clausura. `primitiveFullClosureValue` obtiene el bloque compilado desde otros campos y está condicionado por `<option: #SistaV1BytecodeSet>`. `StackInterpreter` distingue activación de clausuras y clausuras completas. Esto confirma implementaciones disponibles, no que ambas rutas se hayan ejecutado durante las pruebas. ([Proyecto VMMaker](https://source.squeak.org/VMMaker.html)).

La propuesta identifica primitivas por clase y selector, o mediante una cadena, y deja fuera una estandarización general de su implementación. La imagen y el código contemporáneos distinguen primitivas numéricas del núcleo y primitivas externas con nombres de función y módulo. Hay una afinidad con identificar una operación externa por nombre; no es la misma gramática ni el mismo mecanismo de enlace. ([Tabla de primitivas del intérprete](https://github.com/OpenSmalltalk/opensmalltalk-vm/blob/3ea6bd17fabb5aca6fcbd175185d2fccdd50d919/src/spur64.stack/interp.c), [registro de primitivas con nombre](https://github.com/OpenSmalltalk/opensmalltalk-vm/blob/3ea6bd17fabb5aca6fcbd175185d2fccdd50d919/mkNamedPrims.sh)).

Algo semejante ocurre con los objetos indexados. El documento elimina las clases de palabras y describe referencias indexadas o bytes, aunque admite acceso a unidades de distintos tamaños. Spur representa formatos indexados de punteros y de datos de diferentes anchos, además de métodos compilados. Por eso su tabla de formatos debe leerse como contrato de la VM inspeccionada, no como derivación directa de las dos alternativas de 1987. ([Formatos y primitivas en el intérprete Spur](https://github.com/OpenSmalltalk/opensmalltalk-vm/blob/3ea6bd17fabb5aca6fcbd175185d2fccdd50d919/src/spur64.stack/interp.c)).

VMMaker contiene `SpurMemoryManager>>classWordArray`, `classDoubleByteArray` y `classDoubleWordArray`, además de `isWords:`. Para Spur de 64 bits, `wordIndexableFormat` lleva a `sixtyFourBitIndexableFormat`, cuyo valor leído es `9`. Ese número es un código interno de formato, no sintaxis del lenguaje. **La consulta de `StackInterpreter defaultObjectMemoryClass` devolvió `NewObjectMemory`**: disponer de clases Spur en la imagen no demuestra que esa configuración de intérprete las use.

El compilador decide el análisis y la generación de código; la VM ejecuta instrucciones y aplica contratos de objetos y primitivas; la imagen define protocolos de recuperación. Esta separación evita atribuir al JIT una regla léxica o atribuir al parser un comportamiento de retorno de la VM.

## Los literales y los archivos requieren distinguir sintaxis de transporte

El documento propone que cada elemento de un array literal tenga la misma forma que un literal independiente, incluyendo `nil`, `true`, `false` y símbolos prefijados con `#`. Cuis aceptó `#(nil true false #foo 'bar' $a)` y `#'a b'`. También aceptó las abreviaturas `#(foo)` y `#(foo (bar))`, evaluadas como `#(#foo)` y `#(#foo #(#bar))`. **Estas abreviaturas difieren de la regla estricta propuesta**, que exige formas de literales independientes. Los ejemplos no constituyen una gramática completa.

Las llaves que 1987 reserva para extensiones tienen un uso actual observado: `{1. 2 + 3}` devolvió `#(1 5)`, un array construido con expresiones evaluadas. La asignación histórica con `_` también se aceptó: `| x | x _ 3. x` devolvió `3`. Son resultados locales del compilador Cuis, no características inferidas del formato de objetos de la VM.

La propuesta permite ampliar ASCII con caracteres que una implementación clasifica como espacios, letras o gráficos. Eso deja decisiones abiertas que no resuelve una afirmación general de “soporte Unicode”. La documentación de Cuis especifica archivos **UTF-8 con finales de línea LF** para el trabajo con GitHub. Eso no demuestra que el scanner admita cualquier carácter Unicode dentro de un identificador. Esa propiedad no quedó verificada mediante MCP. ([Cuis y GitHub](https://github.com/Cuis-Smalltalk/Cuis-Smalltalk-Dev/blob/master/Documentation/CuisAndGitHub.md)).

El formato de archivo también es una cuestión separada. El documento define *chunks* terminados en `!` y un protocolo de lectura especial mediante `scanFrom:`. La existencia actual de otros formatos de intercambio no exige que cambien los mensajes, bloques o expresiones del lenguaje.

**Tonel es un formato externo de archivos**: distribuye definiciones de paquetes, clases y métodos, en lugar de constituir por sí solo una nueva sintaxis de expresiones Smalltalk. Su utilidad para versionar fuentes responde a otra capa de la propuesta. Es necesario comparar el lector y la representación de clases, además del cuerpo de los métodos. ([Especificación y herramientas de Tonel](https://github.com/pharo-vcs/tonel)).

La definición de clases de 1987 mediante `Behavior newSuperclass:instanceVariables:classVariables:poolDictionaries:` también es una propuesta de interfaz, no una descripción neutral de todos los constructores de clases posteriores. Los formatos de archivo y los constructores de una imagen deben documentarse por dialecto.

## Tres inconsistencias del archivo necesitan una edición crítica

La copia del repositorio presenta inconsistencias internas. **No se las atribuye al original histórico**: falta un facsímil o una segunda copia independiente que permita distinguir erratas de 1987, problemas de transcripción o conversión posterior.

La producción `string` usa comillas dobles como delimitador, igual que `comment`. Sin embargo, los ejemplos de strings y categorías usan comillas simples. La gramática copiada y los ejemplos no describen una única distinción inequívoca entre string y comentario.

La producción `number` separa los dígitos de `fraction-and-exponent` como alternativas, en lugar de concatenarlos. Tal como está escrita, no deriva `1.0e3`, aunque la tabla lo da como válido. Esto impide usar las ecuaciones copiadas como especificación ejecutable sin una corrección explícita.

Finalmente, `block-constructor` exige `block-declarations`, y sus alternativas no incluyen vacío. Así no deriva el bloque común `[1]`. La prosa y los usos de bloques sugieren la necesidad de una alternativa opcional, pero eso sigue siendo una interpretación que necesita cotejo documental.

La revisión mantiene intacto [Smalltalk-1987-syntax.md](../Smalltalk-1987-syntax.md). Una corrección futura debería preservar la lectura recibida, explicar cada enmienda y citar la copia histórica que la respalda. Cambiar silenciosamente las ecuaciones mezclaría preservación documental con una nueva especificación.

## Qué usar al diseñar o portar Smalltalk en 2026

El documento sirve como lista de decisiones y como evidencia de problemas que sus autores intentaban resolver. Para números, reflection y retornos no locales conviene escribir ejemplos ejecutables del dialecto de destino. Para primitivas y formatos de objetos hace falta fijar además la revisión de VM; para archivos hay que identificar el lector o serializador concreto.

Esta revisión no establece compatibilidad general, conformidad ANSI ni corrección del JIT. Deja un contraste reproducible y acotado: hay coincidencias importantes en bloques y control, diferencias numéricas medidas y protocolos de VM que no siguen literalmente la propuesta. Las pruebas pendientes están registradas para ampliar esa comparación sin convertir una observación local en una afirmación sobre todos los Smalltalks.

Las fuentes con referencias `master` o `Cog` se consultaron el 30 de septiembre de 2026 y pueden cambiar. Los enlaces al código generado de VM fijan el commit `3ea6bd17fabb5aca6fcbd175185d2fccdd50d919`; no se estableció que ese commit sea exactamente la versión del paquete VMMaker cargado en la imagen viva. La etiqueta `4.7.0 (Cog)` proviene del literal que devuelve `VMMaker class>>versionString`, no identifica una revisión exacta del paquete ni la configuración de la VM ejecutora.
