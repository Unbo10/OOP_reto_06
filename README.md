# Reto 6

Para este reto se nos solicitaba agregarles excepciones a los retos [1](https://github.com/Unbo10/OOP_reto_01) y [5](https://github.com/Unbo10/OOP_reto_05), que correspondían a, respectivamente, cinco sub-retos en los que se aplicaban conceptos de programación funcional para solucionar tareas básicas usando ciclos, funciones y tipos de datos básicos; y la creación de un módulo paquete [``Shape``](https://github.com/Unbo10/OOP_reto_05/tree/main/Shape) que contenía tres subpaquetes más, [``Edge``](https://github.com/Unbo10/OOP_reto_05/tree/main/Shape/Edge), [``Rectangle``](https://github.com/Unbo10/OOP_reto_05/tree/main/Shape/Rectangle) y [``Triangle``](https://github.com/Unbo10/OOP_reto_05/tree/main/Shape/Triangle). Todos contenían módulos archivo que incluían la definición de la clase respectiva así como métodos para ejecutar una prueba con y sin entrada del usuario (en caso de que fuesen ejecutados como script activos).

A continuación se muestran y explican las implementaciones de excepciones en ambos retos. Cabe aclarar que, para practicar el uso de las ramas en Git y no copiar código de un repositorio a otro, se subieron los cambios a una rama denominada ``excepctions`` en ambos repositorios. Sin embargo, en la explicación a continuación se mostrarán los links que facilitarán la localización de cada rama.

# Tabla de contenido

- [Reto 1](#reto1)
  - [``one.py``](#one)
  - [``two.py``](#two)
  - [``three.py``](#three)
  - [``four.py``](#four)
  - [``five.py``](#five)
- [Reto 5](#reto5)
  - [Implementación multipaquete en ``Shape``](#multi)
    - [``vertex.py``](#vertex)
    - [``edge.py``](#edge)
    - [``rectangle.py``](#rectangle)
    - [``square.py``](#square)
    - [``triangle.py``](#triangle)
    - [``equilateral.py``](#equilateral)
    - [``isosceles.py``](#isosceles)
    - [``Triangle``/rectang.pyle](#trirectangle)
    - [``scalene.py``](#scalene)
    - [``shape.py``](#shape)
  - [Implementación en módulo único](#AllinOne)
- [Conclusión](#conclusion)

<h2 id = "reto1"> Excepciones en el <a href = "https://github.com/Unbo10/OOP_reto_01/tree/exceptions">reto 1</a></h2>

<h3 id = "one"> <code><a href = "https://github.com/Unbo10/OOP_reto_01/blob/exceptions/one.py">one.py</a></code> </h3>

> Cree una función que realice operaciónes básicas (suma, resta, multiplicación, división) entre dos números, según la elección del usuario. La entrada de la función serán dos operandos y el caracter a usar para la operación.

Por un lado, se definió la clase ``OperationTypeError`` que hereda de ``ValueError``. Esto ya que se consideró que ``ValueError`` es la clase padre de excepciones que ocurren cuando la entrada concuerda con el tipo de variable que la almacena, pero la misma no es apropiada en el contexto. Así,

```python
class OperationTypeError (ValueError):
    def __init__(self, *args: object) -> None:
        super().__init__("The input must be a single character containing one of the four operations: \'+\', \'-\', \'*\' or \'/\'")
```

Por otro lado, se crearon las funciones ``enter_number`` y ``enter_operation``. La primera recibe un argumento entero que indica cuál de los dos números se está ingresando (el primero o el segundo), rectificando que se ingrese un número real (de punto flotante); mientras que la segunda no recibe ningún argumento y se encarga de verificar que el usuario haya ingresado uno de las cuatro posibles operaciones. En ambos casos se maneja la excepción respectiva y un ciclo ``while`` para que el usuario pueda reingresar un valor correcto:

```python
def enter_number (iteration: int) -> float:
    number_order: str = ""
    if iteration == 1:
        number_order = "first"
    elif iteration == 2:
        number_order = "second"
    n: float = 0
    while True:
        try:
            n = float(input("Enter the {} number: ".format(number_order)))
            break
        except ValueError as e:
            print("Make sure you are entering a real number")
    return n

def enter_operation() -> chr:
    op: chr = ""
    while True:
        try:
            op = input("Enter the operation you want to perform between the two numbers (+, -, * or /): ")
            if op in ['+', '-', '*', '/']:
                break
            else:
                raise OperationTypeError()
        except OperationTypeError as e:
            for message in e.args:
                print(message)
    return op
```

Por otra parte, la función ``user_input_test`` se creó para facilitar la importación del archivo como módulo y su utilización. En suma, aparte de lo que había previamente en el script, se le agregó un manejo de ocurrencias de ``ZeroDivisionError`` en caso de que se quiera dividir cualquier número por 0:

```python
def user_input_test() -> None:
    print("Hello! This is a program to compute an addition, subtraction, multiplication or division between two real numbers")
    num1: float = enter_number(1)
    num2: float = enter_number(2)
    op: chr = enter_operation()
    try:
        print("The result of the operation {} {} {} is:".format(num1, op, num2), basic_operation(num1, num2, op))
    except ZeroDivisionError as e:
        print("The operation {} / {} is not defined in the real numbers: no real number can be divided by zero".format(num1, num2))
```

Finalmente, **como se añadió en todos los demás archivos**, al ejecutar el programa se llama a la función ``user_input_test`` dentro de un bloque ``try-except-finally``, en el que se maneja cualquier ``KeyboardInterrupt`` para que no se muestre todo el objeto ``traceback`` de la excepción sino solo un mensaje de despedida:

```python
if __name__ == "__main__":
    try:
        user_input_test()
    except KeyboardInterrupt:
        print("\n\nThe program was interrupted by the user.")
    finally:
        print("Goodbye! 👋", end="")
```

<h3 id = "two"> <code><a href = "https://github.com/Unbo10/OOP_reto_01/blob/exceptions/two.py">two.py</a></code> </h3>

> Objetivo: Crear una función que le permita validar si una palabra es palíndroma. Condición: no puede usar ``slicing`` para revertir la palabra y verificar si es la misma que la original.

En primer lugar, el error ``NotWordError`` fue definido de la siguiente manera:

```python
class NotWordError(Exception):
    def __init__(self) -> None:
        super().__init__("The input is not a word, or at least not any set of Spanish or English characters")
```

Es decir, es una excepción que hereda de ``Exception`` (la recomendada para definir excepciones nuevas). Cuando esta ocurre quiere decir que la entrada del usuario no corresponde con una palabra en español o inglés. Para corroborar esto, se creó la función ``enter_word``:

```python
def enter_word() -> str:
    valid_string: bool = False
    word_lower_case: str = ""
    accents: tuple = ["á", "é", "í", "ó", "ú"]
    while not valid_string:
        try:
            word = input("Enter a string: ")
            word_lower_case = word.lower()
            for c in range(len(word_lower_case)):
                if ("a" <= word_lower_case[c] <= "z") or (word_lower_case[c] == "ñ") or (word_lower_case[c] in accents):
                    pass
                else:
                    raise NotWordError()
            valid_string = True
        except NotWordError as e:
            for m in e.args:
                print(m)
    return word
```

Como se puede evidenciar dentro del ciclo ``while`` de la función, se utiliza manejo de excepciones para corroborar que el usuario haya ingresado una palabra válida, y, en caso de no haberlo hecho, que pueda volver a intentarlo tras haberle notificado de ello.

En suma, se creó la función ``user_input_test``, que recopila lo que antes estaba el script a ejecutar. Entonces, en el script simplemente se llama a esta función. Esto se hizo pensando en que este programa podría ser un módulo, por lo que incorporar la función de prueba como parte del mismo y no del script sería más adecuado, asímismo para ejecutar el programa en un bloque ``try-except-finally``.

<h3 id = "three"><a href="https://github.com/Unbo10/OOP_reto_01/blob/exceptions/three.py"> <code>three.py</code> </a></h3>

> Objetivo: Definir una función que reciba una lista de enteros y devuelva solo aquellos que sean primos.

Primeramente, se definieron las excepciones, hijas de ``Exception``, ``NotIntegerError`` y ``NotSingleAlphaError`` para tratar con entradas que no fueran números enteros y para determinar cuándo se desea dejar de ingresar números (en efecto, ``KeyboardInterrupt`` pudo haber sido mejor, pero de esta manera se conserva la lógica original y se experimenta un poco), respectivamente:

```python
class NotIntegerError (Exception):
    def __init__(self, message="The value entered is not an integer."):
        super().__init__(message)

class NotSingleAlphaError (Exception):
    def __init__(self, message="The value entered is not a single non-numeric character nor an integer."):
        super().__init__(message)
```

En segundo lugar, la función ``enter_integer`` fue creada para corroborar que el usuario inserte un entero y no un número de punto flotante (causa de NotIntegerError), múltiples caracteres no numéricos (causa de NotSingleAlphaError) o un solo carácter no numérico (causa de TypeError y de terminación de entrada del usuario):

```python	
def enter_integer() -> bool:
    n: str = input("Enter a number: ")
    if ("." in n) and (len(n) > 1):
        raise NotIntegerError
    elif n.isnumeric() == False:
        if (n[0] != "-"):
            if len(n) == 1:
                raise TypeError 
            else:
                raise NotSingleAlphaError
        elif n[1:].isnumeric() == False:
            raise NotIntegerError
        else:
            return n
    else:
        return n
```

Por último, en la función ``user_input_test`` se interpretan las excepciones que pueden ocurrir en ``enter_integer``:

```python
def user_input_test() -> None:
    int_list = []
    done: bool = False
    n: int = 0
    print("To stop entering numbers, enter a single non-numeric value. This program will return the (integer) prime numbers that you enter.")
    while done == False:
        try:
            n = enter_integer()
        except NotIntegerError as e:
            print(e)
        except NotSingleAlphaError as e:
            print(e)
        except IndexError:
            print("You must enter a value.")
        except TypeError:
            done = True
        else:
            int_list.append(int(n))

    print("The prime numbers in {} are:".format(int_list), filter_primes(int_list))
```

<h3 id = "four"> <code><a href = "https://github.com/Unbo10/OOP_reto_01/blob/exceptions/four.py">four.py</a></code> </h3>

> Objetivo: Crear una función que reciba una lista de enteros y devuelva la mayor suma entre dos elementos consecutivos.

Por un lado, se definió la excepción ``NotValidCharacterError`` de la siguiente manera:

```python
class NotValidCharacterError (Exception):
    def __init__(self, *args: object) -> None:
        super().__init__(*args)
```

Por otro lado, en ``enter_input`` se manejaron las ocurrencias de ``NotValidCharacterError`` cuando no se ingresaba un entero y de ``ValueError`` cuando se detenía el ingreso de enteros:

```python
def user_input_test() -> None:
    int_list = []
    done: bool = False
    n: str = ""
    print("This program will return the greatest sum between two consecutive integer numbers.")
    print("To stop entering numbers, enter the \"?\" character")
    while done == False:
        try:
            n = input("Enter a number: ")
            if n == "?":
                raise ValueError
            elif (n.isnumeric() == False) and (n[0] != "-"):
                raise NotValidCharacterError("The value entered is not a single non-numeric character nor an integer.")
            elif (n.isnumeric() == False) and (n[1:].isnumeric() == False):
                raise NotValidCharacterError("The value entered is not a single non-numeric character nor an integer.")
        except ValueError:
            done = True
        except NotValidCharacterError as e:
            print(e)
        else:
            int_list.append(int(n))
    print("The greatest consecutive sum between two elements in {} is:".format(int_list), find_greatest_consecutive_sum(int_list))
```

<h3 id = "five"> <code><a href = "https://github.com/Unbo10/OOP_reto_01/blob/exceptions/five.py">five.py</a></code> </h3>

Por un lado, se definió la excepción ``NotWordError`` así:

```python
class NotWordError (Exception):
    def __init__(self, *args: object) -> None:
        super().__init__(*args)
```

Por otro lado, en ``user_input_test`` se mejoró el manejo de excepción usado en [``three.py``](#three) usando la función integrada ``is.alpha()`` para verificar si la entrada del usuario es una palabra. Tras haber probado, esta función también cosidera tildes y virgulillas, por lo que no había necesidad de hacerlo manualmente. En suma, se utilizó ``KeyboardInterrupt`` para detener la entrada de palabras:

```python
def user_input_test() -> None:
    word_list = []
    done: bool = False
    word: str = ""
    print("This program will return the words that have matching characters in the list.")
    print("To stop entering words, press Ctrl+C.")
    while done == False:
        try:
            word = input("Enter a word: ")
            if word.isalpha() == False:
                raise NotWordError("The value entered is not a word.")
        except NotWordError as e:
            print(e)
        except KeyboardInterrupt:
            print()
            done = True
        else:
            word_list.append(word)

    print("The words with matching characters in {} are:".format(word_list), find_anagrams(word_list))
```

<h2 id = "reto5"> Excepciones en el <a href = "https://github.com/Unbo10/OOP_reto_05/tree/exceptions"> reto 5 </a> </h2>

<h3 id = "multi"> Implementación multipaquete en <a href = "https://github.com/Unbo10/OOP_reto_05/tree/exceptions/Shape"><code> Shape </code></a> </h3>

Tal como en el reto anterior, si se ejecuta cualquier módulo archivo como script activo se llaman a las funciones ``test_default`` y ``test_user_input`` de las respectivas clases dentro de un bloque ``try-except-finally`` para manejar cualquier ``KeyboardInterrupt`` durante la ejecución:

```python
if __name__ == "__main__":
   try:
      test_default()
      print()
      test_user_input()
   except KeyboardInterrupt:
      print("\n\nProgram stopped by the user", end="")
   finally:
      print("Thank you for using the program!")
```

Además, en todos hay un bloque ``try-except`` que permite la ejecución e importación del módulo al añadir la ruta del directorio padre que contiene a todo el paquete ``Shape``:

```python
from os.path import dirname, realpath
import sys
try:
   grandparent_dir = dirname(dirname(dirname(realpath(__file__))))
   sys.path.index(grandparent_dir)
except ValueError:
   sys.path.append(grandparent_dir)
```

<h4 id = "vertex"> <a href = "https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/Edge/vertex.py"><code>vertex.py</code></a> </h4>

En el módulo vértice se definió el método ``enter_coordinate`` para evitar que el usuario ingresara un valor que no fuese un número real:

```python
def enter_coordinate(coordinate: str) -> float:
   done: bool = False
   while done == False:
      try:
         n = float(input("Enter the {}: ".format(coordinate)))
      except ValueError as e:
         print("The value enterd is not a real number")
      else:
         done = True
   return n
```

Este método se usó en todas las líneas donde el usuario ingresaba coordenadas de vértices de figuras y de lados.

<h4 id = "edge"> <a href = "https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/Edge/edge.py"><code>edge.py</code></a> </h4>

El único cambio fue el manejo de cualquier instancia de ``AssertionError`` que indica que se está intentando insertar una coordenada previamente insertada (esto mismo se hizo en todos los demás módulos).

```python
def test_user_input() -> None:
   ...
   while valid_input == False:
      try:
         x2: float = enter_coordinate("x coordinate of the second vertex")
         y2: float = enter_coordinate("y coordinate of the second vertex")
         if  (x1 == x2) and (y1 == y2):
            raise AssertionError
      except AssertionError as e:
         print("The edge cannot be created because the vertices are the same")
      else:
         valid_input = True
   ...
```

<h4 id = "rectangle"> <a href = "https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/Rectangle/rectangle.py"><code>rectangle.py</code></a> </h4>

Para el caso del módulo rectángulo fue necesario definir una nueva excepción, ``NotStraightEdgeError``:

```python
class NotStraightEdgeError(Exception):
   def __init__(self, message="The rectangle cannot be created because the vertex entered doesn't form a straight edge with the previous vertex") -> None:
      super().__init__(message)
```

La razón es que, para verificar que los vértices que el usuario proporciona crean un rectángulo, se necesita verificar si los lados que forman son rectos. Esto se hizo en la función ``test_user_input`` en un bloque similar al utilizado en el módulo ``edge``:

```python
def test_user_input() -> None:
   ...
   valid_input: bool = False
   v1: Vertex = Vertex(0, 0)
   v2: Vertex = Vertex(0, 0)
   v3: Vertex = Vertex(0, 0)
   v4: Vertex = Vertex(0, 0)

   x1: float = enter_coordinate("x coordinate of the first vertex")
   y1: float = enter_coordinate("y coordinate of the first vertex")
   v1 = Vertex(x1, y1)
   while valid_input == False:
      try:
         x2: float = enter_coordinate("x coordinate of the second vertex")
         y2: float = enter_coordinate("y coordinate of the second vertex")
         if (x1 == x2) and (y1 == y2):
            raise AssertionError
         elif (x1 != x2) and (y1 != y2):
            raise NotStraightEdgeError
      except AssertionError:
         print("The rectangle cannot be created because the vertices are the same")
      except NotStraightEdgeError as e:
         print(e)
      else:
         v2 = Vertex(x2, y2)
         valid_input = True
   
   valid_input = False
   while valid_input == False:
      try:
         x3: float = enter_coordinate("x coordinate of the third vertex")
         y3: float = enter_coordinate("y coordinate of the third vertex")
         if ((x2 == x3) and (y2 == y3)) or ((x1 == x3) and (y1 == y3)):
            raise AssertionError
         elif (x2 != x3) and (y2 != y3):
            raise NotStraightEdgeError
      except AssertionError:
         print("The rectangle cannot be created because the input matches a previous vertex")
      except NotStraightEdgeError as e:
         print(e)
      else:
         v3 = Vertex(x3, y3)
         valid_input = True
   
   valid_input = False
   while valid_input == False:
      try:
         x4: float = enter_coordinate("x coordinate of the fourth vertex")
         y4: float = enter_coordinate("y coordinate of the fourth vertex")
         v4 = Vertex(x4, y4)
         e2: Edge = Edge(v1, v4)
         e1: Edge = Edge(v3, v4)
         if ((x3 == x4) and (y3 == y4)) or ((x2 == x4) and (y2 == y4)) or ((x1 == x4) and (y1 == y4)):
            raise AssertionError
         elif ((e1.slope != None) and (e1.slope != 0)) or ((e2.slope != None) and (e2.slope != 0)):
            raise NotStraightEdgeError
      except AssertionError:
         print("The rectangle cannot be created because the input matches a previous vertex")
      except NotStraightEdgeError as e:
         print(e)
      else:
         valid_input = True
   ...
```

La causa de solicitar al usuario ingresar las coordenadas en el sentido de las manecillas del reloj es para facilitar la verificación de la rectitud de los lados. De otro modo, habrían muchos más casos a considerar y el programa se volvería innecesariamente extenso.

<h4 id = "square"> <a href = "https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/Rectangle/square.py"><code>square.py</code></a> </h4>

Al igual que en el módulo *rectángulo*, la excepción ``NotSquareSide`` se definió dentro del módulo *cuadrado*:

```python
class NotSquareSide (Exception):
   def __init__(self, message="The vertices do not form a side of the square") -> None:
      super().__init__(message)
```

Del mismo modo, sus ocurrencias se administraron en la función ``test_user_input`` dentro de ciclos ``while`` para verificar que los lados que el usuario proporcione tengan la misma longitud y sean perpendiculares con los lados adyacentes:

```python
def test_user_input() -> None:
   ...
   v1: Vertex = Vertex(0, 0)
   v2: Vertex = Vertex(0, 0)
   v3: Vertex = Vertex(0, 0)
   v4: Vertex = Vertex(0, 0)
   length: float = 0
   e1: Edge = Edge(Vertex(0, 0), Vertex(0, 0))
   valid_input: bool = False

   x1: float = enter_coordinate("x coordinate of the first vertex")
   y1: float = enter_coordinate("y coordinate of the first vertex")
   v1 = Vertex(x1, y1)
   while valid_input == False:
      try:
         x2: float = enter_coordinate("x coordinate of the second vertex")
         y2: float = enter_coordinate("y coordinate of the second vertex")
         if (x1 == x2) and (y1 == y2):
            raise AssertionError
         elif (x1 != x2) and (y1 != y2):
            raise NotSquareSide
      except AssertionError:
         print("The square cannot be created because the vertices are the same")
      except NotSquareSide as e:
         print(e)
      else:
         v2 = Vertex(x2, y2)
         e1 = Edge(Vertex(x1, y1), Vertex(x2, y2))
         length = e1.length
         valid_input = True
      
   valid_input = False
   while valid_input == False:
      try:
         x3: float = enter_coordinate("x coordinate of the third vertex")
         y3: float = enter_coordinate("y coordinate of the third vertex")
         e1 = Edge(Vertex(x2, y2), Vertex(x3, y3))
         if (x3 == x1) and (y3 == y1):
            raise AssertionError
         elif (e1.length != length):
            raise NotSquareSide
      except AssertionError:
         print("The square cannot be created because the vertices are the same")
      except NotSquareSide as e:
         print(e)
      else:
         v3 = Vertex(x3, y3)
         valid_input = True
   
   valid_input = False
   while valid_input == False:
      try:
         x4: float = enter_coordinate("x coordinate of the fourth vertex")
         y4: float = enter_coordinate("y coordinate of the fourth vertex")
         e1 = Edge(Vertex(x3, y3), Vertex(x4, y4))
         if ((x4 == x2) and (y4 == y2)) or ((x4 == x1) and (y4 == y1)) or ((x4 == x3) and (y4 == y3)):
            raise AssertionError
         elif (e1.length != length):
            raise NotSquareSide
      except AssertionError:
         print("The square cannot be created because the vertices are the same")
      except NotSquareSide as e:
         print(e)
      else:
         v4 = Vertex(x4, y4)
         valid_input = True
   ...
```

Análogamente a *rectángulo*, se solicitó al usuario ingresar las coordenadas de los vértices de modo que no se formara una diagonal, es decir, que fuesen colineales los vertices, para facilitar la verificación de la rectitud de los lados.

<h4 id = "triangle"> <a href = "https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/Triangle/triangle.py"><code>triangle.py</code></a> </h4>

Tal como en los dos módulos anteriores, en *triángulo* se definió una nueva excepción:

```python
class NotTriangleError(Exception):
   def __init__(self, *args) -> None:
      super().__init__(*args)
```

Sin embargo, a diferencia de los dos módulos previos, no se debe verificar la rectitud de los lados sino simplemente que los tres vértices sean diferentes y que no estén alineados (en caso contrario, se genera una excepción ``NotTriangleError``). Esto se puede ver en las líneas añadidas a ``test_user_input``:

```py
def test_user_input() -> None:
   ...
   e1: Edge = Edge(Vertex(0, 0), Vertex(0, 0))
   e2: Edge = Edge(Vertex(0, 0), Vertex(0, 0))
   valid_input: bool = False

   x1: float = float(input("x coordinate of the first vertex"))
   y1: float = float(input("y coordinate of the first vertex"))
   while valid_input == False:
      try:
         x2: float = float(input("x coordinate of the second vertex"))
         y2: float = float(input("y coordinate of the second vertex"))
         if ((x1 == x2) and (y1 == y2)):
            raise AssertionError
      except AssertionError:
         print("The second vertex cannot be the same as the first vertex")
      else:
         e1 = Edge(Vertex(x1, y1), Vertex(x2, y2))
         valid_input = True
   
   valid_input = False
   while valid_input == False:
      try:
         x3: float = float(input("x coordinate of the third vertex"))
         y3: float = float(input("y coordinate of the third vertex"))
         e2 = Edge(Vertex(x2, y2), Vertex(x3, y3))
         if (e1.slope == e2.slope):
            raise NotTriangleError("The vertices entered do not form a triangle")
      except NotTriangleError as e:
         print(e)
      else:
         valid_input = True
   ...
```

<h4 id = "equilateral"> <a href = "https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/Triangle/equilateral.py"><code>equilateral.py</code></a> </h4>

Por otro lado, en el módulo *equilátero* se definió la excepción ``NotEquilateralError`` como hija de ``NotTriangleError``:

```py
class NotEquilateralError(NotTriangleError):
   def __init__(self, message="The vertices do not form an equilateral triangle") -> None:
      super().__init__(message)
```

Cualquier ocurrencia de esta implicaba que los tres vertices eran colineales o que los lados no tenían la misma longitud, por lo que no podía formarse un triángulo equilátero. Su implementación se puede evidenciar en ``test_user_input``: 

```py
def test_user_input() -> None:
   ...
   e1: Edge = Edge(Vertex(0, 0), Vertex(0, 0))
   e2: Edge = Edge(Vertex(0, 0), Vertex(0, 0))
   e3: Edge = Edge(Vertex(0, 0), Vertex(0, 0))
   length: float = 0
   valid_input: bool = False

   x1: float = enter_coordinate("x coordinate of the first vertex")
   y1: float = enter_coordinate("y coordinate of the first vertex")
   while valid_input == False:
      try:
         x2: float = enter_coordinate("x coordinate of the second vertex")
         y2: float = enter_coordinate("y coordinate of the second vertex")
         if ((x1 == x2) and (y1 == y2)):
            raise AssertionError
      except AssertionError:
         print("The vertices must be different")
      else:
         e1 = Edge(Vertex(x1, y1), Vertex(x2, y2))
         length = round(e1.length, 4)
         valid_input = True
   
   valid_input = False
   while valid_input == False:
      try:
         x3: float = enter_coordinate("x coordinate of the third vertex")
         y3: float = enter_coordinate("y coordinate of the third vertex")
         e2 = Edge(Vertex(x2, y2), Vertex(x3, y3))
         e3 = Edge(Vertex(x3, y3), Vertex(x1, y1))
         if (round(e2.length, 4) != length) or (round(e3.length, 4) != length):
            # print(length, round(e2.length, 4), round(e3.length, 4))
            raise NotEquilateralError
      except NotEquilateralError as e:
         print(e)
      else:
         valid_input = True
   ...
```

Aclarar que las longitudes de los lados se redondean a cuatro para ser comparadas con la misma precisión con la que el usuario ingresó las coordenadas de los vértices.

<h4 id = "isosceles"> <a href = "https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/Triangle/isosceles.py"><code>isosceles.py</code></a> </h4>

Tal como en el módulo *equilátero*, una nueva excepción se definió a partir de ``NotTriangleError`` llamada ``NotIsoscelesError``:

```py
class NotIsoscelesError(NotTriangleError):
   def __init__(self, message="The vertices do not form an isosceles triangle") -> None:
      super().__init__(message)
```

Esta excepción ocurría cuando los vértices estuviesen alineados o cuando no hubieran dos pares de lados con la misma longitud:

```py
def test_user_input() -> None:
   ...
   valid_input: bool = False
   e1: Edge = Edge(Vertex(0, 0), Vertex(0, 0))
   e2: Edge = Edge(Vertex(0, 0), Vertex(0, 0))
   e3: Edge = Edge(Vertex(0, 0), Vertex(0, 0))

   x1: float = enter_coordinate("x coordinate of the first vertex")
   y1: float = enter_coordinate("y coordinate of the first vertex")
   while valid_input == False:
      try:
         x2: float = enter_coordinate("x coordinate of the second vertex")
         y2: float = enter_coordinate("y coordinate of the second vertex")
         if ((x1 == x2) and (y1 == y2)):
            raise AssertionError
      except AssertionError:
         print("The vertices must be different")
      else:
         e1: Edge = Edge(Vertex(x1, y1), Vertex(x2, y2))
         valid_input = True
   
   valid_input = False
   while valid_input == False:
      try:
         x3: float = enter_coordinate("x coordinate of the third vertex")
         y3: float = enter_coordinate("y coordinate of the third vertex")
         e2 = Edge(Vertex(x2, y2), Vertex(x3, y3))
         e3 = Edge(Vertex(x3, y3), Vertex(x1, y1))
         if (e1.length != e2.length) and (e1.length != e3.length) and (e2.length != e3.length):
            raise NotIsoscelesError
         elif (e1.slope == e2.slope) or (e1.slope == e3.slope) or (e2.slope == e3.slope):
            raise NotIsoscelesError
      except NotIsoscelesError as e:
         print(e)
      else:
         valid_input = True
   ...
```

<h4 id = "trirectangle"> <a href = "https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/Triangle/rectangle.py"><code>Triangle/rectangle.py</code></a> </h4>


Siguiendo el patrón, en el módulo del triángulo rectángulo, la excepción ``NotRightTriangleError`` se definió de la siguiente manera:

```py
class NotRightTriangleError(Exception):
   def __init__(self, message="The vertices do not form a right triangle") -> None:
      super().__init__(message)
```

Esta excepción se manejaba en caso de que no se formaran dos catetos y una hipotenusa con los vértices ingresados. Además, los vértices no tenían que ser ingresados en ningún orden en particular, razón por la cual se consideraron tres casos, dos en los que se genere un cateto con los primeros dos vértices y uno en el que el par de vértices ingresados genere la hipotenusa:

```py
def test_user_input() -> None:
   ...
   valid_input: bool = False
   e1: Edge = Edge(Vertex(0, 0), Vertex(0, 0))

   x1: float = enter_coordinate("x coordinate of the first vertex")
   y1: float = enter_coordinate("y coordinate of the first vertex")
   while valid_input == False:
      try:
         x2: float = enter_coordinate("x coordinate of the second vertex")
         y2: float = enter_coordinate("y coordinate of the second vertex")
         if ((x1 == x2) and (y1 == y2)):
            raise AssertionError
      except AssertionError:
         print("The second vertex cannot be the same as the first vertex")
      else:
         valid_input = True
   
   valid_input = False
   while valid_input == False:
      try:
         x3 = float(input("Enter the x coordinate of the third vertex: "))
         y3 = float(input("Enter the y coordinate of the third vertex: "))
         e1 = Edge(Vertex(x1, y1), Vertex(x2, y2))
         if ((x1 == x3) and (y1 == y3)) or ((x2 == x3) and (y2 == y3)):
            raise AssertionError
         elif (x1 == x2):
            if (y3 != y1) and (y3 != y2):
               raise NotRightTriangleError
         elif (y1 == y2):
            if (x3 != x1) and (x3 != x2):
               raise NotRightTriangleError
         else:
            e2 = Edge(Vertex(x2, y2), Vertex(x3, y3))
            e3 = Edge(Vertex(x3, y3), Vertex(x1, y1))
            if (e2.slope != 0 and e2.slope != None) or (e3.slope != 0 and e3.slope != None):
               raise NotRightTriangleError
      except AssertionError:
         print("The third vertex cannot be the same as any of the other two vertices")
      except NotRightTriangleError as e:
         print(e)
      else:
         valid_input = True
   ...
```

<h4 id = "scalene"> <a href = "https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/Triangle/scalene.py"><code>scalene.py</code></a> </h4>

La función ``test_user_input`` del módulo *escaleno* es esencialmente la misma que la del módulo *triángulo* (incluso hereda la misma excepción trabajada en este). Los únicos cambios se encuentran en la sintaxis de las frases a imprimir en consola. Por tanto, no se entrará en detalle sobre lo que se hizo en este módulo.

<h4 id = "shape"> <a href = "https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/shape.py"><code>shape.py</code></a> </h4>

Este es posiblemente el módulo mejor optimizado, ya que usando excepciones se logró eliminar un método dentro de la clase ``Shape`` ([``eliminate_repeated_vertices``](https://github.com/Unbo10/OOP_reto_05/blob/main/Shape/shape.py#L26)) y la forma de detener el ingreso de vértices es mucho más ameno tanto para el usuario como para el programador (usando ``KeyboardInterrupt``). Además, se creó la excepción ``InsufficientVerticesError`` para evitar que el usuario insertara menos de tres vértices (el caso de un triángulo se sobreescribió en la clase ``Triangle`` del módulo ``triangle.py`` usando polimorfismo):

```py	
class InsufficientVerticesError(Exception):
   def __init__(self, message="The shape must have more than three vertices") -> None:
      super().__init__(message)
```

Igualmente, en el caso en que el usuario inserte vértices que no formen una figura convexa se maneja un ``IndexError`` así:

```py
def test_user_input() -> None:
   ...
   given_vertices: list = []
   vertex_type: str = "x"
   user_input = 0
   x: float = 0
   y: float = 0
   conti = True
   print("To stop entering vertices press Ctrl+C")
   while conti == True:
      try:
         x = enter_coordinate("x coordinate of the vertex")
         y = enter_coordinate("y coordinate of the vertex")
         for i in given_vertices:
            if (i.x == x) and (i.y == y):
               raise AssertionError
      except AssertionError:
         print("The vertex has already been entered")
      except KeyboardInterrupt:
         conti = False
      else:
         given_vertices.append(Vertex(x, y))

   try:
      shape1 = Shape(given_vertices)
      if (len(given_vertices)==3):
         raise InsufficientVerticesError
   except InsufficientVerticesError as i:
      print()
      print(i)
   except IndexError:
      print()
      print("The shape is not convex")
   else:
      print("The shape is regular:", shape1._is_regular)
      print("The vertices are: ", end="")
      shape1.get_shape_vertices()
      print("The edges are: ", end="")
      shape1.get_shape_edges()
      print("The inner angles are:", shape1.get_inner_angles())
```

<h3 id = "AllinOne"> Implementación en módulo único </h3>

En el módulo [**``AllinOne.py``**](https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/AllinOne.py) se copiaron todos los cambios hechos en cada uno de los módulos individuales y se creó además la clase ``Test`` que contiene todas las funciones ``test_user_input``. Estas se pueden llamar con un único método de esta misma clase nombrado ``test_user_input_all``. De este modo, el archivo de prueba de este módulo, [``test_AllinOne.py``](https://github.com/Unbo10/OOP_reto_05/blob/exceptions/test_AllinOne.py), se redujo a la importación de la clase ``Test`` y un bloque de excepciones para manejar cualquier ``KeyboardInterrupt`` (tal como se hizo en [``test_multimodules.py``](https://github.com/Unbo10/OOP_reto_05/blame/exceptions/test_multimodules.py)):

```py
from Shape.AllinOne import Test

if __name__ == "__main__":
   try:
      test = Test()
      test.test_user_input_all()
   except KeyboardInterrupt:
      print("\n\nThe program was stopped by the user")
   finally:
      print("Thank you for using the program!", end="")
```

<h2 id = "conclusion"> Conclusión </h2>

Las excepciones son extremadamente útiles, en especial en un lenguaje como Python en el que las excepciones son un método ampliamente usado para el control de flujo. Esto se evidenció en todas las implementaciones de manejo de excepciones hechas en este reto y su variedad (definidas por el programador e integradas no-fatales y fatales). Asímismo, junto con la importación de paquetes se observó el poder de la reutilización de buen código (solo hay que remitirse a la función [``enter_coordinate``](https://github.com/Unbo10/OOP_reto_05/blob/exceptions/Shape/Edge/vertex.py#L18), presente en todos los módulos archivo).

Finalmente, recomendaría a otros programadores investigar más sobre los casos en los que se deben definir excepciones personalizadas, porque si bien facilitan la comprensión del código, personalmente las encontré como una copia de ``Exception`` sin mucho más que el mensaje personalizado. Igualmente, se debe mejorar la definición de las excepciones de manera similar a como se hizo con los módulos de triángulos: heredando.

