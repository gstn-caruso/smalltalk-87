# Propuesta de sintaxis estándar de Smalltalk de 1987

**Edición a cargo de:** L. Peter Deutsch, ParcPlace Systems\
**Colaboradores (orden alfabético, lista parcial):** George Bosworth (Digitalk), Steve Burbeck (Softsmarts), L. Peter Deutsch (ParcPlace Systems), Barry Haynes (Apple Computer), Ralph Johnson (University of Illinois, Urbana-Champaign), J. Eliot B. Moss (University of Massachusetts, Amherst), Dave Thomas (Carleton University), David Ungar (Stanford University), Steve Vegdahl (Tektronix), Allen Wirfs-Brock (Tektronix).

> Copyright (C) 1987 de ParcPlace Systems. Todos los derechos reservados. Nada de lo expuesto en este documento constituye un compromiso de ParcPlace Systems de implementar o dar soporte a ninguna de las prestaciones aquí tratadas.

Este es un documento de trabajo que propone una definición completa de la sintaxis de la versión de 1987 del lenguaje Smalltalk-80 (TM). Este lenguaje es una revisión del lenguaje Smalltalk-80 definido en *Smalltalk-80: The Language and Its Implementation* (Addison-Wesley, 1983), al que en adelante nos referiremos como el Libro Azul.

Este documento pretende definir la sintaxis del lenguaje de manera completa y formal, pero solo trata su semántica de manera parcial e informal. Tenemos la intención de ampliar este documento con una definición formal completa de la semántica del lenguaje en algún momento futuro. También reconocemos que la utilidad del lenguaje Smalltalk-80 depende en gran medida del conjunto de clases y mensajes definidos: en este aspecto se parece a Lisp y se diferencia de la mayoría de los demás lenguajes. Especificamos un conjunto muy pequeño de clases y mensajes que consideramos necesarios para sustentar cualquier sistema Smalltalk-80 útil y un punto de partida adecuado para la estandarización; sin embargo, reconocemos que no hemos tratado este tema de manera suficiente.

Los cambios realizados en cada versión no están marcados en el texto porque dificultan demasiado la lectura de las ecuaciones y tablas: el historial de versiones resume los cambios importantes. Se solicitan comentarios.

Este documento resultará más fácil de leer si se imprime con una fuente de ancho fijo o casi fijo.

Smalltalk-80 es una marca comercial de ParcPlace Systems.

## Historial de versiones

- **[6] 1 de julio de 1987:** Incorpora la carta de presentación de George Bosworth, con algunas adiciones menores y aclaraciones identificadas por Glenn Krasner; traslada los argumentos de bloques y los operadores de asignación a la sintaxis léxica para evitar problemas con los lugares donde pueden aparecer espacios en blanco; cambia las primitivas (otra vez) para especificarlas mediante clase y selector o mediante una cadena. Esta versión se envió a todos los participantes del taller para su aprobación.
- **[5] 5 de junio de 1987:** Incorpora comentarios del taller del 26–27 de febrero y aportes posteriores. Un agradecimiento especial a Steve Vegdahl por depurar la versión anterior de la sintaxis escribiendo un analizador sintáctico para ella.
  - **Sintaxis léxica:** Se aclaró el supuesto de que el analizador léxico siempre toma el token más largo en caso de ambigüedad; se hicieron cambios menores a la sintaxis de los números, incluida la eliminación de los números de punto flotante en bases alternativas y de la notación científica para enteros; se reservaron explícitamente las llaves y el acento grave para futuras extensiones; se cambió el papel de `~` en los selectores binarios.
  - **Otra sintaxis:** Se eliminaron la sintaxis de creación dinámica de arreglos y las expresiones múltiples entre paréntesis; se agregó `#'...'` para denotar un símbolo que contiene caracteres arbitrarios; se restablecieron las clases con variables de instancia tanto con nombre como indexadas, y se cambió la sintaxis de las definiciones de clases; se especificaron las primitivas mediante cadena y número.
  - **Semántica:** Se eliminaron todos los mensajes con significados fijos; se aclaró la posición sobre la redeclaración de nombres.
- **[4] 25 de febrero de 1987:** Se dio a los mensajes de control la semántica completa de los mensajes; se hicieron cambios menores a la sintaxis de los números; se permitieron expresiones múltiples entre paréntesis y se aclaró el papel de `.`; se contemplaron conjuntos de caracteres ampliados y reducidos; se especificaron las primitivas mediante clase y selector. Esta es la versión presentada en el taller de implementadores del 26–27 de febrero.
- **[3] 2 de enero de 1987:** Se trasladó la sintaxis del Libro Azul a un apéndice; se reorganizó el resumen de cambios; se eliminó el punto de extensión para declaraciones; se eliminaron las declaraciones y las expresiones múltiples entre paréntesis; se redujo el tratamiento especial de los mensajes de control; se permitieron tanto números como cadenas para las primitivas; se agregó una sección sobre el formato de archivos; se hicieron cambios menores que reflejan comentarios de la revisión interna. Esta es la versión enviada a Eliot Moss para su distribución a los participantes del taller del 11–12 de diciembre.
- **[2] 22 de diciembre de 1986:** Se agregaron una comparación resumida con el Libro Azul y la sintaxis de definición de clases.
- **[1] 20 de diciembre de 1986:** Primera versión; todavía sin sintaxis para las definiciones de clases.

Las ecuaciones sintácticas se presentan en BNF, extendida con estas construcciones:

```text
[x]  optional x
x*   zero or more occurrences of x
x+   one or more occurrences of x
```

## 1. Resumen de cambios respecto del Libro Azul

Esto es solo un resumen; consultá las siguientes secciones para conocer los detalles.

### Aclaraciones

- Se corrigieron algunos errores menores en la clasificación de caracteres.
- Se aclaró el papel sintáctico de `super`.
- Se definió el orden de evaluación del receptor y los argumentos de izquierda a derecha.
- Se explicitó la sintaxis de las primitivas.
- Se explicitó la sintaxis para definir clases.

### Eliminaciones

- Ya no se ofrecen constantes de punto flotante en bases alternativas ni notación científica para enteros.
- Las clases pueden definirse con instancias que contengan variables de instancia con nombre y/o referencias a objetos indexadas, o únicamente bytes indexados de 8 bits; ya no pueden contener «palabras» (bytes de 16 bits).

### Incompatibilidades

- No pueden aparecer comillas duplicadas dentro de los comentarios. Esto no cambia qué programas se aceptan, sino únicamente cómo se analizan sintácticamente.
- Se cambió la sintaxis de los arreglos literales para que cada elemento tenga exactamente la misma forma que un literal independiente.
- Los argumentos y las variables temporales de los bloques tienen un ámbito correcto tanto léxico como dinámico; no se almacenan en el contexto de origen. Muchos programas existentes no se ejecutarán correctamente, pero pueden modificarse para funcionar tanto con las reglas actuales como con las propuestas.
- La primitiva, si existe, aparece antes de las variables temporales del método, no después.
- Las primitivas estándar se identifican, si es necesario, mediante un nombre de clase y un selector; las primitivas no estándar se identifican mediante una cadena.

### Adiciones

- Se contemplan extensiones del conjunto de caracteres y se reconoce la posibilidad de un conjunto de caracteres reducido.
- Se permiten letras minúsculas dentro de los enteros en bases alternativas.
- Los símbolos literales pueden contener caracteres arbitrarios; la nueva sintaxis para ello es `#` seguido de una cadena.
- Los arreglos literales pueden contener `nil`, `true` y `false`, además de otros literales.
- La secuencia `:=` ahora también significa asignación, como alternativa a `_` (flecha hacia la izquierda).
- Los bloques pueden tener variables temporales.
- No se debe obligar a los mensajes de control (`ifTrue:`, etc.) a recibir bloques explícitos como argumentos. Todos los mensajes, sin excepción, se comportan de la misma manera semánticamente y pueden ser redefinidos por el usuario.
- Las primitivas estándar pueden tener una descripción en forma de cadena además de un número.

## 2. Estándar propuesto de 1987

La siguiente sintaxis está organizada de la misma manera que la sintaxis del Libro Azul presentada en el Apéndice, con una diferencia importante: está diseñada específicamente para ser reconocida por un analizador sintáctico de descenso recursivo sin retroceso y con un búfer de un solo token (como el analizador actual de Smalltalk-80). Las diferencias respecto del Libro Azul se indican con `**`. Las construcciones eliminadas se marcan con `--`; las nuevas, con `++`.

La sintaxis presentada en el Libro Azul no indica dónde se permiten separadores (espacios en blanco o comentarios). En las secciones tituladas **Primitivas léxicas** más abajo, no se permiten separadores entre construcciones; en las demás secciones, pueden aparecer separadores entre cualesquiera dos construcciones (símbolos terminales o no terminales, o agrupaciones).

### Conjunto de caracteres

La sintaxis estándar de Smalltalk de 1987 se basa en el conjunto de caracteres ASCII. Aunque no se menciona explícitamente en las ecuaciones siguientes, todos los caracteres ASCII no imprimibles (códigos 0–31 y 127) se tratan como caracteres de espacio en blanco; todos los demás caracteres se mencionan explícitamente en la siguiente sección sobre primitivas léxicas.

Otros conjuntos de caracteres pueden incluir caracteres no especificados de otra manera, de tres tipos: de espacio en blanco, alfabéticos y gráficos. Los caracteres de espacio en blanco se ignoran en todas partes salvo en las constantes de carácter y de cadena; los caracteres alfabéticos son léxicamente equivalentes a las letras (pueden iniciar identificadores y aparecer dentro de ellos); los caracteres gráficos se permiten en los literales de carácter y de cadena y son inválidos en los demás lugares. En ASCII, todos los caracteres no gráficos con códigos inferiores a 128 (códigos 0–31 y 127) son espacios en blanco; solo las letras, mayúsculas y minúsculas, son alfabéticas. Una implementación que vaya más allá de ASCII puede designar caracteres adicionales como de espacio en blanco, alfabéticos y gráficos de una manera dependiente de la implementación. No se define ningún mecanismo particular para hacerlo.

El conjunto de caracteres ISO difiere de ASCII y redefine ciertos caracteres ASCII para darles un significado distinto, en particular los corchetes y las llaves. Por el momento no se especifican representaciones alternativas para estos elementos del lenguaje; esto deberá abordarse en la versión final del estándar.

### Primitivas léxicas

La sintaxis léxica es formalmente ambigua: por ejemplo, `abc:` puede analizarse como un identificador seguido de un carácter que no sea una comilla, o como una palabra clave. La ambigüedad se resuelve en todos los casos a favor del token más largo que pueda formarse a partir de un punto dado del texto fuente. Por lo tanto, `abc:` siempre se considera una palabra clave si `a` inicia el token. La definición de token se proporciona solo con fines expositivos.

```text
token = number | identifier | special-character | keyword |
        block-argument | assignment-operator | binary-selector |
        character-constant | string

digit = '0' | ... | '9'
digits = digit+
big-digits = (digit | letter)+  "as appropriate for radix"

number = (digits ['r' big-digits] | fraction-and-exponent) |
         fraction-and-exponent
fraction-and-exponent = '.' digits [('e' | 'E') ['-'] digits]

letter = 'A' | ... | 'Z' | 'a' | ... | 'z'
identifier = letter (letter | digit)*

special-character = '+' | '/' | '\\' | '*' | '~' | '<' | '>' |
                    '=' | '@' | '%' | '|' | '&' | '?' | '!' | ','

non-quote-character = digit | letter | special-character |
                      whitespace-character | '[' | ']' | '{' | '}' |
                      '(' | ')' | '_' | '^' | ';' | ':' | '$' | '#'

block-argument = ':' identifier
assignment-operator = '_' | ':='
keyword = identifier ':'

binary-selector = ('-' | special-character)
                  ('~' special-character | special-character)*

character-constant = '$' (non-quote-character | '"' | "'")
symbol = identifier | binary-selector | keyword+
string = '"' (non-quote-character | '"' '"' | "'")* '"'
comment = '"' (non-quote-character | "'")* '"'
separators = (non-printing-character | comment)*
```

La sintaxis de los números en el Libro Azul y su interpretación por el compilador actual de Smalltalk producen algunos resultados inesperados:

| Entrada | Resultado de `doIt` |
|---|---|
| `1e31000` | `1e31000` (`e` como exponente funciona tanto con enteros como con números de punto flotante) |
| `1.0e3` | `1000.0` |
| `16r1e3` | `4096` (la `e` minúscula sigue significando exponente) |
| `16r10.0` | `16.0` (funcionan los números de punto flotante en bases alternativas) |
| `16r1E3` | `483` (la `E` mayúscula significa dígito hexadecimal) |
| `.5` | `5` (el `.` inicial se interpreta como separador de sentencias) |

Con la nueva sintaxis, estos ejemplos tendrían las siguientes interpretaciones:

| Entrada | Resultado |
|---|---|
| `1e3` | Inválido: `e` está reservada para números de punto flotante y requiere un `.` |
| `1.0e3` | `1000.0` |
| `16r1e3` | `483` (sin distinción entre mayúsculas y minúsculas) |
| `16r10.0` | Inválido: no se admiten números de punto flotante en bases alternativas |
| `16r1E3` | `483` |
| `.5` | `0.5` (el `.` inicial indica un número de punto flotante) |

Como se indica en la sintaxis anterior, `e` y `E` están reservadas para números de punto flotante, que también requieren un `.`.

Se corrigieron los errores del Libro Azul y se eliminó el límite de dos caracteres para la longitud de los selectores binarios. El carácter `~` sigue teniendo un papel especial: no puede ser el último carácter de un selector binario (a menos que sea el único), porque de lo contrario expresiones como `x+-5` serían ambiguas (`x + -5` o `x +- 5`).

Se permite `:=` para la asignación, ya que el estándar ASCII tiene un guion bajo en lugar de una flecha hacia la izquierda (`_`). Esto introduce una anomalía sintáctica menor, que se explica en la sección sobre expresiones más abajo. Las llaves y el acento grave se reservan deliberadamente para futuras extensiones del lenguaje.

### Términos atómicos

```text
named-constant = 'nil' | 'true' | 'false'
symbol-constant = '#' (symbol | string)
array-constant = '#' '(' literal* ')'
literal = ['-'] number | named-constant | symbol-constant |
          character-constant | string | array-constant
variable-name = identifier  "other than a named-constant,
                             pseudo-variable-name, or 'super'"
```

La nueva sintaxis para las constantes de arreglo es más sencilla de explicar que la sintaxis actual de Smalltalk-80, pero es diferente y algo más extensa. `nil`, `true` y `false` se reconocen explícitamente como constantes. Se agrega una nueva sintaxis, `#` seguido de una cadena, para escribir constantes de símbolo que contengan caracteres arbitrarios.

### Expresiones y sentencias

```text
primary = variable-name | pseudo-variable-name | literal |
          block-constructor | subexpression
pseudo-variable-name = 'self' | 'thisContext'

unary-message = unary-selector
unary-selector = identifier
binary-message = binary-selector primary unary-message*
keyword-message = (keyword primary unary-message* binary-message*)+

cascaded-messages = (';' (unary-message | binary-message |
                           keyword-message))*
messages = unary-message+ binary-message* [keyword-message] |
           binary-message+ [keyword-message] |
           keyword-message
rest-of-expression = [messages cascaded-messages]

expression = variable-name
               (assignment-operator expression | rest-of-expression) |
             keyword '=' expression  "see below" |
             primary rest-of-expression |
             'super' messages cascaded-messages

expression-list = expression ('.' expression)* ['.']
temporaries = '|' temporary-list '|' | '||'
subexpression = '(' expression ')'
temporary-list = declared-variable-name*
declared-variable-name = variable-name

statements = ['^' expression ['.'] | expression ['.' statements]]
block-constructor = '[' block-declarations statements ']'
block-declarations = temporaries |
                     block-argument+
                       ('|' [temporaries] | '||' temporary-list '|' | '||')
```

Para mantener separados el análisis léxico y el sintáctico y, a la vez, permitir construcciones como `x:=3`, se introduce la alternativa `keyword '=' expression` para la asignación. Debe leerse como si fuera `variable-name ':=' assignment`.

Se reescribió la sintaxis de los mensajes para facilitar su análisis sintáctico y, esperamos, su lectura. Se reconocen explícitamente las pseudovariables y el requisito de que `super` vaya seguido de un mensaje. Se permiten mensajes en cascada a `super`, aunque algunos compiladores existentes de Smalltalk-80 no lo permiten. La sintaxis de `.` permite considerarlo un separador, que puede ir seguido de un elemento vacío al final de una lista, o un terminador, que puede omitirse opcionalmente al final de una lista.

Los bloques pueden declarar variables temporales locales. Esta es la única adición significativa a la sintaxis y la semántica del Libro Azul. Un bloque que tenga tanto argumentos como variables temporales requiere un doble `|` entre los argumentos y las variables temporales. Esto es necesario porque el analizador léxico agrupa los caracteres `|` consecutivos en un único selector binario. Por ejemplo:

```smalltalk
[:x || t | y & z]
```

Sin el doble `|`, esto podría significar «argumento `x`, variable temporal `t`, devolver `y & z`» o «argumento `x`, devolver `t | y & z`». Exigir el doble `|` da lugar a la segunda interpretación sintáctica del separador y distingue sin ambigüedad la declaración de variables temporales.

Se rechazaron otras soluciones consideradas: prohibir `|` como operador binario (es la representación estándar de «o»); eliminar el `|` después de la lista de argumentos (empeora la lectura); o hacer que la distinción dependa de si los nombres después del primer `|` ya están definidos (eso introduciría una dependencia sintáctica sutil respecto de propiedades distantes).

Existe una restricción semántica que no puede expresarse mediante una sintaxis libre de contexto: los nombres de variable válidos deben estar declarados en un ámbito que los contenga. Esto incluye variables globales, de diccionario compartido, de clase o de instancia de la clase donde se define el método o de una superclase; argumentos o variables temporales del método o bloque que los contiene; y variables temporales de una subexpresión que los contiene. Un tratamiento completo de los ámbitos de los nombres excede esta propuesta. Se especifican las siguientes reglas:

- Un nombre local (argumento o variable temporal de un método o bloque) no debe entrar en conflicto con un nombre no local accesible en el mismo ámbito (variable global, de diccionario compartido, de clase o de instancia).
- Un nombre local puede entrar en conflicto con otro nombre local accesible en el mismo ámbito; la declaración interna tiene precedencia.

Se recomienda a los implementadores de compiladores que emitan una advertencia cuando el código redeclare un nombre local. Esto ayuda a detectar la práctica actual de Smalltalk en la que un nombre usado como argumento de un bloque también se declara entre las variables temporales del método, por ejemplo:

```smalltalk
temp |
...
[:temp | ...]
```

### Métodos

```text
message-pattern = unary-selector |
                  binary-selector declared-variable-name |
                  (keyword declared-variable-name)+

primitive = '<' 'primitive:' [primitive-identification] '>'
primitive-identification = symbol symbol | string
method = message-pattern [primitive] [temporaries] statements
```

Las primitivas se identifican mediante una clase y un selector o mediante una cadena. Los primeros identifican primitivas estándar; la interpretación de la segunda no está definida. Si falta la identificación de la primitiva, se usan la clase y el nombre del selector del método que la contiene. En general, no se espera estandarizar las primitivas. Lo que se propone estandarizar es el comportamiento de ciertos mensajes en ciertas clases, independientemente de que se implementen como primitivas.

La implementación (el compilador) es responsable de comprobar que las primitivas se asocien únicamente a métodos y clases para los que sean válidas; esta correspondencia es parte de la definición del lenguaje. Una implementación puede incluir primitivas polimórficas que puedan asociarse válidamente a diversas clases y métodos.

La especificación de la primitiva precede a las variables temporales del método, en lugar de seguirlas. Esto parece más intuitivo, ya que la primitiva se ejecuta antes de establecer las vinculaciones de las variables temporales.

### Clases

El sistema Smalltalk-80 adopta un enfoque bastante distinto para crear y editar clases que para hacerlo con métodos: los métodos se definen mediante una sintaxis textual y una interfaz de mensajes para compilarla, mientras que las operaciones sobre clases se definen como mensajes explícitos que reciben diversos tipos de argumentos en forma de cadena. El Libro Azul no introduce una sintaxis propiamente dicha para las clases; supone que se crean usando los mensajes descritos en el Capítulo 16. Sin embargo, la definición de clases forma parte del lenguaje por derecho propio, al igual que la definición de métodos, por lo que esta propuesta especifica el mensaje fundamental para crear clases. La semántica de modificar clases existentes y lo que ocurre con las instancias existentes cuando se modifica una clase quedan fuera de su alcance.

Crear una clase a partir de una especificación se considera igual que crear cualquier otro objeto, y no guarda relación con instalarla bajo un nombre en algún diccionario. Del mismo modo, la creación de un método compilado está separada de su instalación en una clase.

Una clase se crea mediante el siguiente mensaje:

```smalltalk
Behavior
    newSuperclass: "Behavior | nil"
    instanceVariables: "Array of: Symbol"
    classVariables: "Array of: Symbol"
    poolDictionaries: "Array of: Symbol"
```

Dar un nombre a una clase, instalarla en un diccionario o clasificarla en una organización quedan fuera del alcance de la definición del lenguaje. Se presupone que una interfaz interactiva permite a los usuarios definir clases sin escribir el mensaje anterior y se ocupa de los nombres y la organización cuando corresponde.

La superclase especificada para una clase puede ser otro `Behavior` o `nil`. Esto último es necesario para la clase `Object` y permite crear clases que no sean subclases de `Object`. Esto entraña muchos riesgos: por ejemplo, si una clase de ese tipo no define `printOn:`, es probable que el sistema Smalltalk entre en un ciclo de recursión la primera vez que se intente inspeccionar una instancia de la clase.

La sintaxis de la cadena proporcionada para describir las variables de instancia es:

```text
inst-var-names = declared-variable-name* [indexed-refs] |
                 indexed-bytes
indexed-refs = '*' 'Object'
indexed-bytes = '*' 'Byte'
```

Por lo tanto, una clase puede contener variables de instancia con nombre que almacenen referencias a objetos, variables de instancia indexadas que almacenen referencias a objetos (por ejemplo, `Array`), ambas (por ejemplo, `OrderedCollection`) o información distinta de referencias a objetos (por ejemplo, `ByteArray`). Se prevé que una implementación proporcione diversos tipos de acceso primitivo a objetos de tipo bit, por ejemplo mediante bytes de 8, 16 o 32 bits, o quizá secuencias arbitrarias de bits. Estos objetos se denominan objetos de «bytes» en lugar de objetos de «bits» porque la propuesta no exige que las implementaciones asignen su espacio de almacenamiento en unidades menores que 8 bits.

### Sintaxis de archivos

La forma en que se almacenan los programas Smalltalk en archivos externos se define en el Capítulo 3 (pp. 29–37) del Libro Verde, no en el Libro Azul. Esta propuesta estandariza lo suficiente del formato externo como para que los archivos de programa puedan analizarse sintácticamente incluso en sistemas incapaces de interpretar todo su contenido. En las ecuaciones sintácticas siguientes, los separadores **no** se permiten implícitamente entre elementos; las ecuaciones deben tomarse exactamente como aparecen.

```text
marker = '!'
non-marker = any character except the marker
separators = non-printing-character*
chunk = (non-marker | marker marker)+ marker
special-read-section = marker chunk (separators chunk)* separators marker
program-file = (separators (special-read-section | chunk))* separators
```

La información aparece en un archivo de programa en «fragmentos» terminados por un marcador, con los marcadores internos duplicados. Un fragmento que no esté precedido por un marcador es simplemente una expresión que debe evaluarse. Un fragmento precedido por un marcador indica el inicio de una sintaxis especial: la expresión se evalúa para producir algún tipo de objeto lector o analizador sintáctico, al que luego se envía `scanFrom:` con el propio flujo del archivo como argumento. Se espera que el lector lea y procese fragmentos del archivo hasta encontrar un fragmento vacío. En otras palabras, lo siguiente podría representar el algoritmo para leer un archivo de programa:

```smalltalk
[self skipSeparators.
 self atEnd]
    whileFalse:
        [(self peekFor: $!)
            ifTrue: [(Object evaluate: self nextChunk) scanFrom: self]
            ifFalse: [(Object evaluate: self nextChunk)]]
```

El propósito de la sección de lectura especial es, principalmente, permitir que las clases lean definiciones de métodos sin copiarlas dos veces adicionales (una para analizar los fragmentos y otra para analizarlas como literales de cadena que se pasarán como argumentos). El sistema actual de Smalltalk-80 copia la definición una vez adicional, ya que la lee como fragmento antes de analizarla; esto claramente puede evitarse si se desea.

Como mínimo, un analizador de archivos debe poder identificar definiciones de métodos. Para ello, se propone definir el mensaje `<Behavior> methodReader` de modo que devuelva un objeto cuyo método `scanFrom:` lea y defina métodos para el receptor. Pueden enviarse mensajes adicionales a este objeto sin comprometer su función, por ejemplo:

```smalltalk
!aBehavior methodReader category: 'something'!
```

Mediante esta convención, una implementación puede definir propiedades adicionales para los métodos que se están leyendo sin comprometer la posibilidad general de analizar sintácticamente los archivos fuente.

## Semántica

### Orden de evaluación

Las expresiones de una lista de expresiones se evalúan de izquierda a derecha. Los envíos de mensajes en una cascada se evalúan de izquierda a derecha. El receptor de un mensaje se evalúa antes que los argumentos; los argumentos se evalúan de izquierda a derecha.

### Clases estándar

Las siguientes clases son conceptualmente necesarias para sustentar el lenguaje definido anteriormente; son las clases de los objetos literales:

- `Integer`
- `Float`
- `Symbol`
- `String`
- `Array`
- `Character`
- `Block`
- `True`, `False`, `Nil`
- `Behavior` (para las clases)

Estas clases no necesitan tener estos nombres específicos, ni su funcionalidad tiene que dividirse exactamente de esta manera. Por ejemplo, los enteros podrían implementarse mediante clases separadas `SmallInteger` y `LargeInteger`, o `True` y `False` podrían ser instancias de una única clase `Boolean`. Estos nombres y esta división de la funcionalidad se usan para describir los mensajes estándar en la siguiente sección.

### Mensajes estándar

El lenguaje, tal como se describe, no tiene mensajes con significados fijos. Esto se considera una fortaleza singular de Smalltalk (y de lenguajes relacionados, como los lenguajes Actor de Hewitt); la experiencia ha demostrado la utilidad de este concepto para cosas como los reenviadores transparentes de mensajes. Por otra parte, cualquier lenguaje útil debe proporcionar funciones básicas, como la aritmética y las estructuras de control, y cualquier lenguaje comercialmente viable debe implementar algunas de estas funciones con mucha eficiencia. Por lo tanto, se define un pequeño conjunto de mensajes que se espera que proporcionen todas las implementaciones de Smalltalk-80. La pragmática de estos y otros mensajes se trata en la siguiente sección.

Los mensajes siguientes son el mínimo absoluto propuesto para dar soporte al lenguaje. Las marcas del margen izquierdo remiten a la sección sobre Pragmática y deben ignorarse aquí.

**Aritmética**

```text
P (Integer) + - * / < > <= >= = ~= (Integer, Float)
P (Integer) // \\ (Integer)
P (Integer) / (Float)
P (Float) + - * / < > <= >= = ~= (Integer, Float)
```

**Control**

```text
P (Block) value
P (Block) value: (Object)
F (Block) whileTrue: (Block)
F (Block) whileFalse: (Block)
F (Block) whileTrue
F (Block) whileFalse
F (Block) repeat
P (Integer) to: (Integer) do: (Block)
(Integer) timesRepeat: (Block)
F (True, False) ifTrue: (Block)
F (True, False) ifTrue: (Block) ifFalse: (Block)
F (True, False) ifFalse: (Block)
F (True, False) ifFalse: (Block) ifTrue: (Block)
F (True, False) and: (Block)
F (True, False) or: (Block)
```

**Varios**

```text
F (Object) == (Object)
```

### Contextos

Los contextos son la única área en la que se proponen varios cambios menores en la semántica de Smalltalk. Todos son compatibles con versiones anteriores si se realizan algunos cambios menores en la Imagen Virtual.

El primer cambio se refiere al ámbito y al tiempo de vida de los argumentos de los bloques (y de las variables temporales, que son nuevas). En la definición actual de Smalltalk-80, los argumentos de los bloques se almacenan en el contexto de origen. Esto impide usar bloques de manera recursiva o desde más de un Process y provoca mensajes de error anómalos si se interrumpe un proceso («Bloque ya activo»). En la definición propuesta, los bloques son clausuras en el sentido de Scheme u otros Lisp modernos con ámbito léxico: ejecutar la construcción `[]` crea un `BlockClosure`, que encapsula únicamente el contexto actual (de origen) y el código; invocar un `BlockClosure` crea un `BlockContext`. Una implementación puede optimizar esto siempre que se mantenga la semántica. Por ejemplo, un bloque que no haga referencia a variables de ámbitos externos ni efectúe un retorno podría no necesitar mantener una referencia al ámbito externo. En consecuencia, el depurador podría disponer de menos información; esto se permite explícitamente.

El segundo cambio se refiere a `^`. Cuando el control retorna de un método pero el contexto al que se retorna es anómalo (por ejemplo, ya se ha retornado de él), el sistema actual envía `cannotReturn: theValue` al contexto desde el que se retorna. La propuesta cambia esto para que el sistema envíe `resumeWith: theValue` al objeto al que se retorna (presumiblemente, aunque no necesariamente, un contexto). Para facilitar la implementación de estructuras de control no estándar, este mensaje debería definirse como primitiva en la clase `Context`.

El tercer cambio también afecta a `^`. Actualmente, `^` desde dentro de un bloque invoca un algoritmo complejo sobre el que los usuarios no tienen control. La propuesta define `^` dentro de un bloque como el envío del mensaje `thisContext remoteReturn: theValue`. Normalmente, este mensaje se definirá en la clase `BlockContext` como una primitiva que ejecute el algoritmo incorporado actual, aunque una evolución futura del sistema para incorporar manejo de excepciones con protección durante el desenrollado de la pila podría afectar la definición. Las implementaciones pueden prohibir retornos a contextos cuya cadena de emisores termine en un lugar distinto de la raíz del proceso actual; estas situaciones deberían manejarse usando explícitamente `resumeWith:`.

Como se indica más abajo, los compiladores podrían evitar la creación de bloques en ciertas circunstancias, como en el mensaje condicional estándar `ifTrue:ifFalse:`. En consecuencia, `thisContext` sería entonces un contexto externo en lugar del contexto actual real (textual). La propuesta exige una implementación absolutamente fiel, lo que puede requerir construir un contexto real para un mensaje condicional si aparece un `thisContext` dentro de una de las alternativas.

### Pragmática

Se exige una semántica de mensajes absolutamente uniforme, pero debe permitirse una implementación más eficiente de los mensajes cuyo significado tenga muy pocas probabilidades de cambiar. Por lo tanto, se permite que ciertos mensajes, aun conservando la misma semántica que todos los demás, tengan una pragmática sustancialmente diferente:

- Ciertos mensajes, si se envían a receptores cuyas clases no estén en un conjunto especificado, pueden ejecutarse mucho más lentamente.
- Ciertos mensajes, si se definen en clases nuevas, o se redefinen o se eliminan sus definiciones en clases existentes, pueden sufrir una penalización sustancial de rendimiento para algunas o todas las clases receptoras. A cambio, en circunstancias normales estos mensajes se ejecutan sustancialmente más rápido que otros.

En la lista anterior de mensajes estándar, los mensajes marcados con `P` tienen la primera propiedad pragmática (`P` indica que el conjunto de clases para ese mensaje está parcialmente fijado); los mensajes marcados con `F` tienen ambas propiedades (`F` indica que el conjunto de clases para ese mensaje debe permanecer completamente fijado para evitar una pérdida de rendimiento).

«Cambiar una definición» significa que la nueva definición no es operacionalmente equivalente a la anterior. Una condición suficiente, pero no necesaria, para comprobarlo es que la nueva definición se compile al mismo código objeto que la anterior. Se recomienda a los implementadores usar una comprobación de este tipo para que, por ejemplo, cambiar los nombres de variables o los comentarios no se considere un cambio de la definición. Las consecuencias pragmáticas de no compilar `ifTrue:ifFalse:` en línea son tan graves que debería darse al usuario la oportunidad de confirmar que eso era realmente lo que pretendía. Esta es una cuestión de interfaz de usuario, no de definición del lenguaje. Ningún compilador existente de Smalltalk-80 conocido por los autores maneja correctamente esta posibilidad.

La excepción `notBoolean`, que resulta de condiciones no booleanas en la definición actual de Smalltalk-80, no cumple con el estándar propuesto. Si una implementación usa internamente un mecanismo como un mensaje `notBoolean`, debe convertirlo automáticamente en un mensaje correcto `ifTrue:ifFalse:` (o equivalente), con dos bloques apropiados como argumentos, sin intervención del usuario.

Los compiladores actuales de Smalltalk-80 que adoptan el concepto de «selectores aritméticos especiales» del Libro Azul no manejan correctamente la redefinición de estos mensajes; esto no cumple con el estándar propuesto.

Nada en este estándar impide que una implementación agregue o elimine mensajes o clases receptoras de las listas anteriores. Estas diferencias pueden estar incorporadas en la implementación o bajo el control del usuario con un compilador suficientemente sofisticado. Dado que se exige que las mejoras mantengan la semántica sin cambios, no se especifican aquí.

### Comprobación estática

Proporcionar receptores o argumentos de una clase incorrecta a los mensajes anteriores casi con certeza producirá un error en tiempo de ejecución. Los compiladores pueden optar por emitir advertencias si consideran que el usuario ha escrito un programa que probablemente produzca un error. Por ejemplo, un programador poco familiarizado con la sintaxis de Smalltalk podría escribir:

```smalltalk
a < b ifTrue: trueStuff ifFalse: falseStuff
```

en lugar de:

```smalltalk
a < b ifTrue: [trueStuff] ifFalse: [falseStuff]
```

Un compilador podría razonablemente pedir confirmación al usuario si un argumento de `ifTrue:ifFalse:` no es un bloque escrito explícitamente. Sin embargo, el estándar propuesto exige que todos los compiladores acepten compilar programas que contengan construcciones cuestionables de este tipo. La mayoría de los compiladores actuales de Smalltalk-80 no lo hacen.

### Contextos

Todos los compiladores existentes de Smalltalk-80 dan un tratamiento especial a algunos o a todos los mensajes de control, compilándolos de una manera que evita crear objetos Context durante la ejecución. Dado que el estándar del lenguaje no especifica nada sobre la creación de contextos, un compilador puede compilar **cualquier** mensaje de una manera que evite crear Contexts, siempre que la semántica del mensaje no se vea afectada. Dado que los Contexts son visibles para el programador en el metanivel, los programas en el metanivel deben estar preparados para la posibilidad de que un envío de mensaje dado en el nivel del código fuente no cree un Context en el nivel de los objetos.

## Apéndice: El Libro Azul

La siguiente sintaxis es la que aparece en la guarda del Libro Azul, ligeramente reorganizada. Se intercalan algunas notas sobre errores y omisiones. Este material se reproduce con permiso de Xerox Corporation.

### Primitivas léxicas

```text
digit = '0' | ... | '9'
digits = digit+
number = [digits 'r'] ['-'] digits [',' digits] ['e' ['-'] digits]
letter = 'A' | ... | 'Z' | 'a' | ... | 'z'
identifier = letter (letter | digit)*
special-character = '+' | '/' | '\\' | '*' | '~' | '<' | '>' |
                    '=' | '@' | '%' | '|' | '&' | '?' | '!' | ','
character = digit | letter | special-character |
            '[' | ']' | '{' | '}' | '(' | ')' | '_' | '^' | ';' | ':' | '$' | '#'
keyword = identifier ':'
unary-selector = identifier
binary-selector = '-' | special-character [special-character]
character-constant = '$' (character | '"' | "'")
symbol = identifier | binary-selector | keyword+
string = '"' (character | '"' '"' | "'")* '"'
comment = '"' (character | '"' | "'")* '"'
separators = (non-printing-character | comment)*
```

La sintaxis de `special-character` y `character` tiene varios errores. `!` aparece en ambos, pero debería aparecer solo en `special-character`; `,` aparece en `character`, pero debería aparecer en `special-character`; `.` no aparece en ninguno, pero debería aparecer en `character`; `-` y el acento grave no aparecen en ninguno, pero deberían aparecer en `special-character`.

No parece haber una buena razón para limitar la longitud de los selectores binarios a dos caracteres. Al parecer, `-` recibe un tratamiento particular por su papel especial al indicar números negativos.

### Términos atómicos

```text
symbol-constant = '#' symbol
array = '(' (number | symbol | string | character-constant | array)* ')'
array-constant = '#' array
literal = number | symbol-constant | character-constant | string | array-constant
variable-name = identifier
```

### Expresiones y sentencias

```text
primary = variable-name | literal | block | '(' expression ')'
unary-object-description = primary | unary-expression
binary-object-description = unary-object-description | binary-expression
unary-expression = unary-object-description unary-selector
binary-expression = binary-object-description binary-selector unary-object-description
keyword-expression = binary-object-description ':' (keyword binary-object-description)+
message-expression = unary-expression | binary-expression | keyword-expression
cascaded-message-expression = message-expression
                             (';' (unary-selector |
                                   binary-selector unary-object-description |
                                   (keyword binary-object-description)+))+
expression = (variable-name '_')*
             (primary | message-expression | cascaded-message-expression)
statements = ['^' expression ['.'] | expression ['.' statements]]
block = '[' [ (':' variable-name)+ '|' statements | statements ] ']'
```

### Métodos

```text
temporaries = '|' variable-name* '|'
message-pattern = unary-selector | binary-selector variable-name |
                  (keyword variable-name)+
method = message-pattern [temporaries [statements] | statements]
```

El Libro Azul omite la sintaxis para indicar métodos primitivos. Esto puede ser deliberado.
