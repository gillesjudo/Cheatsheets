### Flowchart Shapes and Mermaid.js Cheatsheet

Here's a cheatsheet for common flowchart shapes and their corresponding Mermaid.js syntax.

#### 1. Process/Action

This shape represents a single step or action in a process.

* **Shape:** Rectangle
* **Mermaid.js Code:** `id[Text inside the rectangle]`
* **Example:**
    ```mermaid
    graph TD
        A[This is a Process Step]
    ```

#### 2. Start/End

This shape indicates the beginning or end of a flowchart.

* **Shape:** Rounded Rectangle or Oval
* **Mermaid.js Code:** `id(Text inside the rounded rectangle)`
* **Example:**
    ```mermaid
    graph TD
        Start(Start) --> Stop(End)
    ```

#### 3. Decision

This shape represents a point in the process where a decision must be made. It typically has two or more paths leading out of it, often labeled "Yes/No" or "True/False."

* **Shape:** Diamond
* **Mermaid.js Code:** `id{Text inside the diamond}`
* **Example:**
    ```mermaid
    graph TD
        A{Is it Raining?}
        A -- Yes --> B[Take an Umbrella]
        A -- No --> C[Leave the Umbrella]
    ```

#### 4. Subroutine/Predefined Process

This shape represents a named process or a block of code that is defined elsewhere. It's used to simplify a complex flowchart by referring to another flowchart or module.

* **Shape:** Rectangle with double vertical lines
* **Mermaid.js Code:** `id[[Text inside the subroutine]]`
* **Example:**
    ```mermaid
    graph TD
        A[[Perform a Subroutine]] --> B[Continue Process]
    ```

#### 5. Input/Output

This shape represents data being input into or output from the process.

* **Shape:** Parallelogram
* **Mermaid.js Code:** `id[/Text inside the parallelogram/]`
* **Example:**
    ```mermaid
    graph TD
        A[/Get User Input/] --> B[Process Data]
    ```

#### 6. Data Storage/Database

This shape represents a step that involves storing or retrieving data from a database.

* **Shape:** Cylinder
* **Mermaid.js Code:** `id[(Text inside the cylinder)]`
* **Example:**
    ```mermaid
    graph TD
        A[Perform Action] --> B[(Access Database)]
    ```

#### 7. Arrow/Connector

These are not shapes themselves but are used to show the flow between shapes.

* **Shape:** Arrow
* **Mermaid.js Code:** `-->`
* **Example:**
    ```mermaid
    graph TD
        A[Step 1] --> B[Step 2]
    ```