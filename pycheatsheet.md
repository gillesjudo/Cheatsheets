# Python Cheatsheet 🐍

---

### 1. Basics

* **Variables:** No explicit type declaration.
    ```python
    x = 10
    name = "Python"
    is_active = True
    ```
* **Comments:** Use a hashtag (`#`) for single-line comments.
    ```python
    # This is a single-line comment
    ```
* **Indentation:** Python uses whitespace for code blocks (e.g., `if` statements, `for` loops, functions).
* **Print:** Output to the console.
    ```python
    print("Hello, World!")
    print(f"The value of x is {x}") # f-string formatting
    ```

---

### 2. Data Types & Structures

* **Strings:** Immutable sequence of characters.
    ```python
    my_string = "Hello"
    first_char = my_string[0] # 'H'
    substring = my_string[1:4] # 'ell'
    ```
* **Lists:** Mutable, ordered sequence of items.
    ```python
    my_list = [1, 2, 3, "a"]
    my_list.append(4) # [1, 2, 3, "a", 4]
    my_list.pop() # removes the last element
    my_list[0] = 99 # [99, 2, 3, "a", 4]
    ```
* **Tuples:** Immutable, ordered sequence of items.
    ```python
    my_tuple = (1, 2, 3)
    # my_tuple[0] = 99 # This will cause an error
    ```
* **Dictionaries:** Mutable, unordered collection of key-value pairs.
    ```python
    my_dict = {"name": "Alice", "age": 30}
    print(my_dict["name"]) # 'Alice'
    my_dict["age"] = 31
    my_dict["city"] = "New York"
    ```
* **Sets:** Mutable, unordered collection of unique items.
    ```python
    my_set = {1, 2, 3, 3, 4} # {1, 2, 3, 4}
    my_set.add(5)
    my_set.remove(1)
    ```

---

### 3. Control Flow

* **If/Elif/Else:** Conditional execution.
    ```python
    x = 10
    if x > 10:
        print("x is greater than 10")
    elif x == 10:
        print("x is 10")
    else:
        print("x is less than 10")
    ```
* **For Loop:** Iterate over a sequence.
    ```python
    for i in range(5): # 0 to 4
        print(i)

    fruits = ["apple", "banana", "cherry"]
    for fruit in fruits:
        print(fruit)
    ```
* **While Loop:** Repeat as long as a condition is true.
    ```python
    count = 0
    while count < 3:
        print(count)
        count += 1
    ```

---

### 4. Functions

* **Defining a Function:**
    ```python
    def greet(name):
        """This function greets the person passed in as a parameter."""
        print(f"Hello, {name}!")
    ```
* **Calling a Function:**
    ```python
    greet("Bob")
    ```
* **Return Values:**
    ```python
    def add(a, b):
        return a + b
    
    result = add(5, 3) # 8
    ```
* **Lambda Functions:** Small anonymous functions.
    ```python
    square = lambda x: x * x
    print(square(5)) # 25
    ```

---

### 5. Object-Oriented Programming (OOP)

* **Classes:** Blueprint for creating objects.
    ```python
    class Dog:
        def __init__(self, name, age):
            self.name = name
            self.age = age

        def bark(self):
            return "Woof!"

    my_dog = Dog("Fido", 3)
    print(f"{my_dog.name} is {my_dog.age} years old.")
    print(my_dog.bark())
    ```
* **Inheritance:**
    ```python
    class Labrador(Dog):
        def __init__(self, name, age, color):
            super().__init__(name, age)
            self.color = color

        def swim(self):
            return "Swimming!"
    ```

---

### 6. File I/O

* **Reading a file:**
    ```python
    with open("my_file.txt", "r") as f:
        content = f.read()
        print(content)
    ```
* **Writing to a file:**
    ```python
    with open("new_file.txt", "w") as f:
        f.write("This is a new line.")
    ```

---

### 7. List Comprehensions

* **A concise way to create lists.**
    ```python
    # Simple list comprehension
    squares = [x*x for x in range(5)] # [0, 1, 4, 9, 16]

    # With a conditional
    even_squares = [x*x for x in range(10) if x % 2 == 0] # [0, 4, 16, 36, 64]
    ```

---

### 8. Common Modules

* **`os`:** Interact with the operating system.
* **`sys`:** System-specific parameters and functions.
* **`math`:** Mathematical functions.
* **`random`:** Generate random numbers.
* **`json`:** Work with JSON data.
* **`requests`:** Make HTTP requests.