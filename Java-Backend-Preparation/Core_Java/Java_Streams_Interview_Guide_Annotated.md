# Java Streams — Coding Interview Questions & Answers

> Pure coding questions on Java Streams — the kind commonly asked in interviews. Each question has a ready-to-run code solution.

---

## Setup — Common Classes Used Throughout This Document

Most questions below use this simple `Employee` class (and sometimes a `Product` class). Keep this in mind while reading each solution.

```java
class Employee {
    int id;
    String name;
    String department;
    double salary;
    int age;
    String city;

    Employee(int id, String name, String department, double salary, int age, String city) {
        this.id = id;
        this.name = name;
        this.department = department;
        this.salary = salary;
        this.age = age;
        this.city = city;
    }

    // getters
    public int getId() { return id; }
    public String getName() { return name; }
    public String getDepartment() { return department; }
    public double getSalary() { return salary; }
    public int getAge() { return age; }
    public String getCity() { return city; }

    public String toString() {
        return name + "(" + department + "," + salary + ")";
    }
}

List<Employee> employees = Arrays.asList(
    new Employee(1, "Ravi", "IT", 55000, 28, "Chennai"),
    new Employee(2, "Priya", "HR", 48000, 32, "Mumbai"),
    new Employee(3, "Aman", "IT", 62000, 25, "Chennai"),
    new Employee(4, "Sara", "Finance", 70000, 40, "Delhi"),
    new Employee(5, "Vikram", "IT", 62000, 35, "Mumbai"),
    new Employee(6, "Neha", "HR", 45000, 29, "Delhi"),
    new Employee(7, "Karan", "Finance", 70000, 30, "Chennai")
);
```

---

## Table of Contents
- [Section A: Sorting Questions](#section-a-sorting-questions)
- [Section B: Nth Highest / Lowest Value Questions](#section-b-nth-highest--lowest-value-questions)
- [Section C: Grouping Questions](#section-c-grouping-questions)
- [Section D: Filtering Questions](#section-d-filtering-questions)
- [Section E: Aggregation (Sum, Average, Count, Max, Min)](#section-e-aggregation-sum-average-count-max-min)
- [Section F: Mapping & Collecting Questions](#section-f-mapping--collecting-questions)
- [Section G: Distinct, Duplicate & Set Questions](#section-g-distinct-duplicate--set-questions)
- [Section H: String-Based Stream Questions](#section-h-string-based-stream-questions)
- [Section I: Array & Number Stream Questions](#section-i-array--number-stream-questions)
- [Section J: Advanced/Tricky Stream Questions](#section-j-advancedtricky-stream-questions)

---

## Section A: Sorting Questions

### Q1. Sort employees by salary in ascending order.
```java
List<Employee> sorted = employees.stream()
        .sorted(Comparator.comparingDouble(Employee::getSalary))
        .collect(Collectors.toList());
sorted.forEach(System.out::println);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`sorted()`** → Sorts stream elements.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n log n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q2. Sort employees by salary in descending order.
```java
List<Employee> sorted = employees.stream()
        .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
        .collect(Collectors.toList());
sorted.forEach(System.out::println);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`sorted()`** → Sorts stream elements.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n log n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q3. Sort employees by department first, then by salary within each department.
```java
List<Employee> sorted = employees.stream()
        .sorted(Comparator.comparing(Employee::getDepartment)
                .thenComparing(Employee::getSalary))
        .collect(Collectors.toList());
sorted.forEach(System.out::println);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`sorted()`** → Sorts stream elements.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n log n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q4. Sort a list of Strings by their length.
```java
List<String> names = Arrays.asList("Ravi", "Priya", "Sam", "Alexandra");
List<String> sorted = names.stream()
        .sorted(Comparator.comparingInt(String::length))
        .collect(Collectors.toList());
System.out.println(sorted);   // [Sam, Ravi, Priya, Alexandra]
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`sorted()`** → Sorts stream elements.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n log n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


---

## Section B: Nth Highest / Lowest Value Questions

### Q5. Find the employee with the highest salary.
```java
Optional<Employee> highest = employees.stream()
        .max(Comparator.comparingDouble(Employee::getSalary));
highest.ifPresent(System.out::println);   // Sara(Finance,70000.0) or Karan (both are 70000 - max returns one)
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`max()`** → Finds the maximum value.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q6. Find the employee with the LOWEST salary.
```java
Optional<Employee> lowest = employees.stream()
        .min(Comparator.comparingDouble(Employee::getSalary));
lowest.ifPresent(System.out::println);   // Neha(HR,45000.0)
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`min()`** → Finds the minimum value.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q7. Find the employee with the 2nd highest salary. (Most commonly asked question!)
```java
Optional<Employee> secondHighest = employees.stream()
        .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
        .distinct()
        .skip(1)
        .findFirst();
secondHighest.ifPresent(System.out::println);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`sorted()`** → Sorts stream elements.
- **`distinct()`** → Removes duplicate elements.
- **`skip()`** → Skips the first N elements.
- **`findFirst()`** → Returns the first matching element.

#### ⏱️ Complexity
- **Time:** O(n log n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---

**Explanation:** Sort employees by salary DESCENDING, then SKIP the first one (the highest), and take the next one. `distinct()` here is a safety net in case objects are literally duplicated — see Q8 for handling duplicate SALARY VALUES correctly.

### Q8. Find the employee with the 2nd highest UNIQUE salary (careful with duplicate salary values).
**Answer:** The tricky part: in our data, Sara and Karan BOTH earn 70000. If we just `skip(1)` after sorting, we might accidentally get another 70000-earner instead of the true "2nd highest DISTINCT salary." The fix is to get DISTINCT SALARY VALUES first, then pick the employee(s) matching the 2nd value.
```java
List<Double> distinctSalariesDesc = employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .collect(Collectors.toList());

double secondHighestSalary = distinctSalariesDesc.get(1);   // 2nd distinct salary value

List<Employee> result = employees.stream()
        .filter(e -> e.getSalary() == secondHighestSalary)
        .collect(Collectors.toList());
System.out.println(result);   // employees with the 2nd highest DISTINCT salary
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`map()`** → Transforms each element into another value.
- **`sorted()`** → Sorts stream elements.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`distinct()`** → Removes duplicate elements.

#### ⏱️ Complexity
- **Time:** O(n log n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q9. Find the Nth highest salary (generic, works for any N).
```java
int n = 3;   // change this to whichever rank you need
List<Double> distinctSalariesDesc = employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .collect(Collectors.toList());

if (distinctSalariesDesc.size() >= n) {
    double nthSalary = distinctSalariesDesc.get(n - 1);
    List<Employee> result = employees.stream()
            .filter(e -> e.getSalary() == nthSalary)
            .collect(Collectors.toList());
    System.out.println(result);
}
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`map()`** → Transforms each element into another value.
- **`sorted()`** → Sorts stream elements.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`distinct()`** → Removes duplicate elements.

#### ⏱️ Complexity
- **Time:** O(n log n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q10. Find the top 3 highest paid employees.
```java
List<Employee> top3 = employees.stream()
        .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
        .limit(3)
        .collect(Collectors.toList());
top3.forEach(System.out::println);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`sorted()`** → Sorts stream elements.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`limit()`** → Keeps only the first N elements.

#### ⏱️ Complexity
- **Time:** O(n log n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q11. Find the employee with the highest salary IN EACH department.
```java
Map<String, Optional<Employee>> highestPerDept = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.maxBy(Comparator.comparingDouble(Employee::getSalary))
        ));
highestPerDept.forEach((dept, emp) -> System.out.println(dept + " -> " + emp.get()));
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


---

## Section C: Grouping Questions

### Q12. Group employees by department.
```java
Map<String, List<Employee>> byDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment));
byDept.forEach((dept, list) -> System.out.println(dept + " -> " + list));
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q13. Count the number of employees in each department.
```java
Map<String, Long> countByDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment, Collectors.counting()));
System.out.println(countByDept);   // {IT=3, HR=2, Finance=2}
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q14. Group employees by department, and get only their NAMES (not the full object).
```java
Map<String, List<String>> namesByDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment,
                Collectors.mapping(Employee::getName, Collectors.toList())));
System.out.println(namesByDept);   // {IT=[Ravi, Aman, Vikram], HR=[Priya, Neha], Finance=[Sara, Karan]}
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q15. Group employees by department, and find the AVERAGE salary in each department.
```java
Map<String, Double> avgSalaryByDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment,
                Collectors.averagingDouble(Employee::getSalary)));
System.out.println(avgSalaryByDept);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q16. Group employees by department, and find the TOTAL salary in each department.
```java
Map<String, Double> totalSalaryByDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment,
                Collectors.summingDouble(Employee::getSalary)));
System.out.println(totalSalaryByDept);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q17. Group employees by city, then by department (multi-level grouping).
```java
Map<String, Map<String, List<Employee>>> byCityThenDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getCity,
                Collectors.groupingBy(Employee::getDepartment)));
System.out.println(byCityThenDept);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q18. Partition employees into two groups: those earning above 60000, and those below.
```java
Map<Boolean, List<Employee>> partitioned = employees.stream()
        .collect(Collectors.partitioningBy(e -> e.getSalary() > 60000));
System.out.println("Above 60000: " + partitioned.get(true));
System.out.println("60000 or below: " + partitioned.get(false));
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`partitioningBy()`** → Splits elements into true and false groups.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---

**Note:** `partitioningBy` always creates exactly 2 groups (`true` and `false`), unlike `groupingBy` which can create many groups.

---

## Section D: Filtering Questions

### Q19. Find all employees working in the "IT" department.
```java
List<Employee> itEmployees = employees.stream()
        .filter(e -> e.getDepartment().equals("IT"))
        .collect(Collectors.toList());
itEmployees.forEach(System.out::println);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q20. Find all employees whose salary is greater than 50000.
```java
List<Employee> highEarners = employees.stream()
        .filter(e -> e.getSalary() > 50000)
        .collect(Collectors.toList());
highEarners.forEach(System.out::println);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q21. Find all employees older than 30, working in "IT" or "Finance."
```java
List<Employee> result = employees.stream()
        .filter(e -> e.getAge() > 30)
        .filter(e -> e.getDepartment().equals("IT") || e.getDepartment().equals("Finance"))
        .collect(Collectors.toList());
result.forEach(System.out::println);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q22. Check if ANY employee earns more than 65000.
```java
boolean anyHighEarner = employees.stream()
        .anyMatch(e -> e.getSalary() > 65000);
System.out.println(anyHighEarner);   // true
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`anyMatch()`** → Returns true if any element matches.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q23. Check if ALL employees earn more than 40000.
```java
boolean allAboveMinimum = employees.stream()
        .allMatch(e -> e.getSalary() > 40000);
System.out.println(allAboveMinimum);   // true
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`allMatch()`** → Returns true if all elements match.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q24. Check if NONE of the employees are below 20 years old.
```java
boolean noneUnderage = employees.stream()
        .noneMatch(e -> e.getAge() < 20);
System.out.println(noneUnderage);   // true
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`noneMatch()`** → Returns true if no elements match.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q25. Find the FIRST employee from the "HR" department.
```java
Optional<Employee> firstHR = employees.stream()
        .filter(e -> e.getDepartment().equals("HR"))
        .findFirst();
firstHR.ifPresent(System.out::println);   // Priya(HR,48000.0)
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`findFirst()`** → Returns the first matching element.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


---

## Section E: Aggregation (Sum, Average, Count, Max, Min)

### Q26. Calculate the total salary of all employees.
```java
double totalSalary = employees.stream()
        .mapToDouble(Employee::getSalary)
        .sum();
System.out.println(totalSalary);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`mapToDouble()`** → Converts elements into a DoubleStream.
- **`sum()`** → Calculates the sum.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q27. Calculate the average salary of all employees.
```java
OptionalDouble avgSalary = employees.stream()
        .mapToDouble(Employee::getSalary)
        .average();
System.out.println(avgSalary.getAsDouble());
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`mapToDouble()`** → Converts elements into a DoubleStream.
- **`average()`** → Calculates the average.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q28. Count the total number of employees.
```java
long count = employees.stream().count();
System.out.println(count);   // 7
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`count()`** → Counts the elements.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q29. Find the maximum and minimum age among employees.
```java
OptionalInt maxAge = employees.stream().mapToInt(Employee::getAge).max();
OptionalInt minAge = employees.stream().mapToInt(Employee::getAge).min();
System.out.println("Max age: " + maxAge.getAsInt());
System.out.println("Min age: " + minAge.getAsInt());
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`mapToInt()`** → Converts elements into an IntStream.
- **`max()`** → Finds the maximum value.
- **`min()`** → Finds the minimum value.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q30. Get all statistics (count, sum, min, max, average) about employee salaries in ONE go.
```java
DoubleSummaryStatistics stats = employees.stream()
        .mapToDouble(Employee::getSalary)
        .summaryStatistics();
System.out.println("Count: " + stats.getCount());
System.out.println("Sum: " + stats.getSum());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Average: " + stats.getAverage());
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`mapToDouble()`** → Converts elements into a DoubleStream.
- **`summaryStatistics()`** → Returns count, sum, min, max and average together.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q31. Calculate the total salary using `reduce()` instead of `sum()`.
```java
double total = employees.stream()
        .map(Employee::getSalary)
        .reduce(0.0, Double::sum);
System.out.println(total);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`map()`** → Transforms each element into another value.
- **`reduce()`** → Combines all elements into one result.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q32. Find the employee with the maximum salary, using `reduce()`.
```java
Optional<Employee> maxSalaryEmp = employees.stream()
        .reduce((e1, e2) -> e1.getSalary() > e2.getSalary() ? e1 : e2);
maxSalaryEmp.ifPresent(System.out::println);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`reduce()`** → Combines all elements into one result.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


---

## Section F: Mapping & Collecting Questions

### Q33. Get a list of only employee NAMES.
```java
List<String> names = employees.stream()
        .map(Employee::getName)
        .collect(Collectors.toList());
System.out.println(names);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`map()`** → Transforms each element into another value.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q34. Get a list of all employee names in UPPERCASE.
```java
List<String> upperNames = employees.stream()
        .map(Employee::getName)
        .map(String::toUpperCase)
        .collect(Collectors.toList());
System.out.println(upperNames);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`map()`** → Transforms each element into another value.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q35. Convert the employee list into a `Map<Integer, String>` of ID to Name.
```java
Map<Integer, String> idToName = employees.stream()
        .collect(Collectors.toMap(Employee::getId, Employee::getName));
System.out.println(idToName);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q36. Convert the employee list into a `Map<String, Double>` of Name to Salary.
```java
Map<String, Double> nameToSalary = employees.stream()
        .collect(Collectors.toMap(Employee::getName, Employee::getSalary));
System.out.println(nameToSalary);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q37. Join all employee names into a single comma-separated String.
```java
String allNames = employees.stream()
        .map(Employee::getName)
        .collect(Collectors.joining(", "));
System.out.println(allNames);   // Ravi, Priya, Aman, Sara, Vikram, Neha, Karan
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`map()`** → Transforms each element into another value.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`joining()`** → Joins strings together.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q38. Join all employee names with a prefix and suffix, like "[Ravi, Priya, ...]".
```java
String result = employees.stream()
        .map(Employee::getName)
        .collect(Collectors.joining(", ", "[", "]"));
System.out.println(result);   // [Ravi, Priya, Aman, Sara, Vikram, Neha, Karan]
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`map()`** → Transforms each element into another value.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`joining()`** → Joins strings together.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q39. Convert a list of employees into a Set of unique department names.
```java
Set<String> departments = employees.stream()
        .map(Employee::getDepartment)
        .collect(Collectors.toSet());
System.out.println(departments);   // [IT, HR, Finance] (order not guaranteed with HashSet)
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`map()`** → Transforms each element into another value.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


---

## Section G: Distinct, Duplicate & Set Questions

### Q40. Get a list of unique department names (no duplicates), keeping insertion order.
```java
List<String> uniqueDepts = employees.stream()
        .map(Employee::getDepartment)
        .distinct()
        .collect(Collectors.toList());
System.out.println(uniqueDepts);   // [IT, HR, Finance]
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`map()`** → Transforms each element into another value.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`distinct()`** → Removes duplicate elements.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q41. Find employees who share the SAME salary as another employee (find duplicate salary values).
```java
Map<Double, Long> salaryCounts = employees.stream()
        .collect(Collectors.groupingBy(Employee::getSalary, Collectors.counting()));

List<Double> duplicateSalaries = salaryCounts.entrySet().stream()
        .filter(entry -> entry.getValue() > 1)
        .map(Map.Entry::getKey)
        .collect(Collectors.toList());

System.out.println(duplicateSalaries);   // [70000.0] - Sara and Karan both earn this

List<Employee> employeesWithDuplicateSalary = employees.stream()
        .filter(e -> duplicateSalaries.contains(e.getSalary()))
        .collect(Collectors.toList());
System.out.println(employeesWithDuplicateSalary);   // [Sara, Karan]
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`map()`** → Transforms each element into another value.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q42. Remove duplicate integers from a list.
```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 4, 4, 5);
List<Integer> unique = numbers.stream()
        .distinct()
        .collect(Collectors.toList());
System.out.println(unique);   // [1, 2, 3, 4, 5]
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`distinct()`** → Removes duplicate elements.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q43. Find duplicate integers in a list (numbers that appear more than once).
```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 4, 4, 4, 5);
Set<Integer> seen = new HashSet<>();
List<Integer> duplicates = numbers.stream()
        .filter(n -> !seen.add(n))   // add() returns false if it was already present
        .distinct()
        .collect(Collectors.toList());
System.out.println(duplicates);   // [2, 4]
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`distinct()`** → Removes duplicate elements.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


---

## Section H: String-Based Stream Questions

### Q44. Check whether a String is a palindrome, using Streams.
```java
String input = "madam";
String reversed = new StringBuilder(input).reverse().toString();
boolean isPalindrome = input.equals(reversed);
System.out.println(isPalindrome);   // true

// Pure-stream style character comparison:
boolean isPalindromeStream = IntStream.range(0, input.length() / 2)
        .allMatch(i -> input.charAt(i) == input.charAt(input.length() - 1 - i));
System.out.println(isPalindromeStream);   // true
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`allMatch()`** → Returns true if all elements match.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q45. Find the FIRST non-repeated character in a String.
```java
String input = "swiss";
Optional<Character> firstNonRepeated = input.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(c -> c, LinkedHashMap::new, Collectors.counting()))
        .entrySet().stream()
        .filter(entry -> entry.getValue() == 1)
        .map(Map.Entry::getKey)
        .findFirst();
System.out.println(firstNonRepeated.orElse(null));   // 'w'
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`map()`** → Transforms each element into another value.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.
- **`findFirst()`** → Returns the first matching element.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q46. Count the occurrences of each character in a String.
```java
String input = "programming";
Map<Character, Long> charCount = input.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(c -> c, Collectors.counting()));
System.out.println(charCount);   // {a=1, g=2, i=1, m=2, n=1, o=1, p=1, r=2}
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q47. Count the number of vowels in a String.
```java
String input = "Java Streams are powerful";
long vowelCount = input.toLowerCase().chars()
        .filter(c -> "aeiou".indexOf(c) != -1)
        .count();
System.out.println(vowelCount);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`filter()`** → Keeps only elements matching the condition.
- **`count()`** → Counts the elements.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q48. Reverse each word in a sentence, but keep the word order the same.
```java
String sentence = "Java Streams are cool";
String result = Arrays.stream(sentence.split(" "))
        .map(word -> new StringBuilder(word).reverse().toString())
        .collect(Collectors.joining(" "));
System.out.println(result);   // avaJ smaertS era looc
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`map()`** → Transforms each element into another value.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`joining()`** → Joins strings together.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q49. Find the longest word in a sentence.
```java
String sentence = "Java Streams make coding much more enjoyable";
Optional<String> longestWord = Arrays.stream(sentence.split(" "))
        .max(Comparator.comparingInt(String::length));
System.out.println(longestWord.get());   // enjoyable
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`max()`** → Finds the maximum value.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q50. Count how many words start with a specific letter (e.g., "s" or "S").
```java
String sentence = "Streams simplify some string based stream solutions";
long count = Arrays.stream(sentence.split(" "))
        .filter(word -> word.toLowerCase().startsWith("s"))
        .count();
System.out.println(count);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`count()`** → Counts the elements.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


---

## Section I: Array & Number Stream Questions

### Q51. Find the sum of all numbers in an int array.
```java
int[] numbers = {1, 2, 3, 4, 5};
int sum = Arrays.stream(numbers).sum();
System.out.println(sum);   // 15
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`sum()`** → Calculates the sum.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q52. Find the maximum and minimum value in an int array.
```java
int[] numbers = {5, 3, 8, 1, 9, 2};
int max = Arrays.stream(numbers).max().getAsInt();
int min = Arrays.stream(numbers).min().getAsInt();
System.out.println("Max: " + max + ", Min: " + min);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`max()`** → Finds the maximum value.
- **`min()`** → Finds the minimum value.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q53. Find all EVEN numbers from a list, and square each of them.
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);
List<Integer> evenSquares = numbers.stream()
        .filter(n -> n % 2 == 0)
        .map(n -> n * n)
        .collect(Collectors.toList());
System.out.println(evenSquares);   // [4, 16, 36, 64]
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`map()`** → Transforms each element into another value.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q54. Generate the first 10 Fibonacci numbers using `Stream.iterate()`.
```java
Stream.iterate(new int[]{0, 1}, f -> new int[]{f[1], f[0] + f[1]})
        .limit(10)
        .map(f -> f[0])
        .forEach(System.out::println);
// 0 1 1 2 3 5 8 13 21 34
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`map()`** → Transforms each element into another value.
- **`limit()`** → Keeps only the first N elements.
- **`iterate()`** → Generates stream elements repeatedly.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q55. Generate a Stream of the first 5 even numbers, using `Stream.iterate()`.
```java
Stream.iterate(2, n -> n + 2)
        .limit(5)
        .forEach(System.out::println);
// 2 4 6 8 10
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`limit()`** → Keeps only the first N elements.
- **`iterate()`** → Generates stream elements repeatedly.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q56. Check whether a number is prime, using Streams.
```java
int number = 29;
boolean isPrime = number > 1 && IntStream.rangeClosed(2, (int) Math.sqrt(number))
        .noneMatch(i -> number % i == 0);
System.out.println(isPrime);   // true
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`noneMatch()`** → Returns true if no elements match.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q57. Print all prime numbers between 1 and 50, using Streams.
```java
IntStream.rangeClosed(2, 50)
        .filter(num -> IntStream.rangeClosed(2, (int) Math.sqrt(num)).allMatch(i -> num % i != 0))
        .forEach(System.out::println);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`filter()`** → Keeps only elements matching the condition.
- **`allMatch()`** → Returns true if all elements match.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q58. Find the factorial of a number, using `reduce()`.
```java
int n = 5;
long factorial = IntStream.rangeClosed(1, n)
        .asLongStream()
        .reduce(1, (a, b) -> a * b);
System.out.println(factorial);   // 120
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`reduce()`** → Combines all elements into one result.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q59. Convert an `int[]` array into a `List<Integer>`.
```java
int[] arr = {1, 2, 3, 4, 5};
List<Integer> list = Arrays.stream(arr)
        .boxed()
        .collect(Collectors.toList());
System.out.println(list);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`boxed()`** → Converts primitive values to wrapper objects.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q60. Merge two lists into one, removing duplicates.
```java
List<Integer> list1 = Arrays.asList(1, 2, 3, 4);
List<Integer> list2 = Arrays.asList(3, 4, 5, 6);
List<Integer> merged = Stream.concat(list1.stream(), list2.stream())
        .distinct()
        .collect(Collectors.toList());
System.out.println(merged);   // [1, 2, 3, 4, 5, 6]
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`distinct()`** → Removes duplicate elements.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


---

## Section J: Advanced/Tricky Stream Questions

### Q61. Find the department with the highest total salary expense.
```java
Map<String, Double> totalByDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment, Collectors.summingDouble(Employee::getSalary)));

Map.Entry<String, Double> topDept = totalByDept.entrySet().stream()
        .max(Map.Entry.comparingByValue())
        .orElseThrow();
System.out.println(topDept.getKey() + " -> " + topDept.getValue());
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.
- **`max()`** → Finds the maximum value.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q62. Find the name of the youngest employee in each department.
```java
Map<String, String> youngestPerDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment,
                Collectors.collectingAndThen(
                        Collectors.minBy(Comparator.comparingInt(Employee::getAge)),
                        opt -> opt.map(Employee::getName).orElse("N/A")
                )));
System.out.println(youngestPerDept);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`map()`** → Transforms each element into another value.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q63. Sort a Map of department-to-average-salary by value, in descending order.
```java
Map<String, Double> avgSalaryByDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment, Collectors.averagingDouble(Employee::getSalary)));

LinkedHashMap<String, Double> sortedMap = avgSalaryByDept.entrySet().stream()
        .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
        .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue,
                (e1, e2) -> e1, LinkedHashMap::new));
System.out.println(sortedMap);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`sorted()`** → Sorts stream elements.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n log n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q64. Find the second lowest salary among employees (without duplicates).
```java
List<Double> distinctSalariesAsc = employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted()
        .collect(Collectors.toList());

double secondLowest = distinctSalariesAsc.get(1);
System.out.println(secondLowest);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`map()`** → Transforms each element into another value.
- **`sorted()`** → Sorts stream elements.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`distinct()`** → Removes duplicate elements.

#### ⏱️ Complexity
- **Time:** O(n log n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q65. Group employees by department, and within each group, sort them by salary descending.
```java
Map<String, List<Employee>> result = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment,
                Collectors.collectingAndThen(Collectors.toList(),
                        list -> {
                            list.sort(Comparator.comparingDouble(Employee::getSalary).reversed());
                            return list;
                        })));
System.out.println(result);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q66. Find employees whose name starts with a vowel.
```java
List<Employee> result = employees.stream()
        .filter(e -> "AEIOU".indexOf(Character.toUpperCase(e.getName().charAt(0))) != -1)
        .collect(Collectors.toList());
result.forEach(System.out::println);   // Aman
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q67. Convert a `List<Employee>` into a `Map<String, List<String>>` — department to list of employee names.
```java
Map<String, List<String>> deptToNames = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment,
                Collectors.mapping(Employee::getName, Collectors.toList())));
System.out.println(deptToNames);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q68. Find duplicate elements between two lists (common elements / intersection).
```java
List<Integer> list1 = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> list2 = Arrays.asList(3, 4, 5, 6, 7);

List<Integer> common = list1.stream()
        .filter(list2::contains)
        .collect(Collectors.toList());
System.out.println(common);   // [3, 4, 5]
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`collect()`** → Collects the result into a List, Set, Map, etc.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q69. Flatten a `List<List<Integer>>` into a single `List<Integer>` using `flatMap()`.
```java
List<List<Integer>> nestedList = Arrays.asList(
        Arrays.asList(1, 2, 3),
        Arrays.asList(4, 5),
        Arrays.asList(6, 7, 8, 9)
);
List<Integer> flatList = nestedList.stream()
        .flatMap(List::stream)
        .collect(Collectors.toList());
System.out.println(flatList);   // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`flatMap()`** → Flattens nested collections.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q70. Find the total number of characters across all employee names combined.
```java
int totalChars = employees.stream()
        .mapToInt(e -> e.getName().length())
        .sum();
System.out.println(totalChars);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`mapToInt()`** → Converts elements into an IntStream.
- **`sum()`** → Calculates the sum.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q71. Check if there are any duplicate employee IDs in the list.
```java
boolean hasDuplicates = employees.stream()
        .map(Employee::getId)
        .distinct()
        .count() < employees.size();
System.out.println(hasDuplicates);   // false, since our IDs are all unique
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`map()`** → Transforms each element into another value.
- **`distinct()`** → Removes duplicate elements.
- **`count()`** → Counts the elements.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q72. Convert a comma-separated String into a sorted list of unique words.
```java
String input = "banana,apple,orange,apple,banana,grape";
List<String> sortedUnique = Arrays.stream(input.split(","))
        .distinct()
        .sorted()
        .collect(Collectors.toList());
System.out.println(sortedUnique);   // [apple, banana, grape, orange]
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`sorted()`** → Sorts stream elements.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`distinct()`** → Removes duplicate elements.

#### ⏱️ Complexity
- **Time:** O(n log n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q73. Find employees who earn ABOVE the average salary of all employees.
```java
double avgSalary = employees.stream()
        .mapToDouble(Employee::getSalary)
        .average()
        .orElse(0);

List<Employee> aboveAverage = employees.stream()
        .filter(e -> e.getSalary() > avgSalary)
        .collect(Collectors.toList());
aboveAverage.forEach(System.out::println);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`mapToDouble()`** → Converts elements into a DoubleStream.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`average()`** → Calculates the average.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q74. Create a frequency map of department names (how many employees per department, but as a percentage).
```java
long total = employees.size();
Map<String, Double> percentageByDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment, Collectors.counting()))
        .entrySet().stream()
        .collect(Collectors.toMap(Map.Entry::getKey, e -> (e.getValue() * 100.0) / total));
System.out.println(percentageByDept);
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`groupingBy()`** → Groups elements based on a key.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


### Q75. Find the employee(s) who have the SAME salary as the MAXIMUM salary (handle ties correctly).
```java
double maxSalary = employees.stream()
        .mapToDouble(Employee::getSalary)
        .max()
        .orElse(0);

List<Employee> topEarners = employees.stream()
        .filter(e -> e.getSalary() == maxSalary)
        .collect(Collectors.toList());
System.out.println(topEarners);   // [Sara(Finance,70000.0), Karan(Finance,70000.0)] - both, correctly!
```

#### 🧠 Approach
Create a stream, apply the required intermediate operations, then use a terminal operation to produce the final result.

#### 🔧 Methods Used
- **`stream()`** → Creates a Stream from the collection.
- **`filter()`** → Keeps only elements matching the condition.
- **`mapToDouble()`** → Converts elements into a DoubleStream.
- **`collect()`** → Collects the result into a List, Set, Map, etc.
- **`max()`** → Finds the maximum value.

#### ⏱️ Complexity
- **Time:** O(n)
- **Space:** Usually O(n)

#### 💡 Interview Notes
- Explain why each Stream method is used.
- Mention whether the operation is intermediate or terminal.
- Discuss duplicate handling and edge cases if applicable.

---


---

## Quick Reference: Common Stream Methods Used Above

| Method | What it does |
|--------|----------------|
| `filter()` | Keeps only elements matching a condition |
| `map()` | Transforms each element into something else |
| `flatMap()` | Flattens nested collections into a single stream |
| `sorted()` | Sorts elements (natural order, or with a Comparator) |
| `distinct()` | Removes duplicate elements |
| `limit(n)` | Keeps only the first `n` elements |
| `skip(n)` | Skips the first `n` elements |
| `collect()` | Gathers stream results into a List/Set/Map/String |
| `reduce()` | Combines all elements into a single result |
| `count()` | Counts the number of elements |
| `anyMatch()` / `allMatch()` / `noneMatch()` | Checks conditions across elements |
| `findFirst()` / `findAny()` | Gets a single element from the stream |
| `max()` / `min()` | Finds the largest/smallest element |
| `groupingBy()` | Groups elements by a key |
| `partitioningBy()` | Splits elements into `true`/`false` groups |
| `joining()` | Combines Strings together with a separator |
| `summaryStatistics()` | Gets count/sum/min/max/average in one call |

---

**These 75 questions cover almost every Stream pattern that shows up in real interviews.** Practice by changing the `employees` list yourself, adding more entries with tricky ties (same salary, same age), and re-running each solution to see how it behaves.
