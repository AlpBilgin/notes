# cheatsheet

## Data Types, variables

### primitive types
```python
myintvar = 5
myfloatvar = 3.0
mystringvar = "Hello"
```

#### print() interpolation
```python
print("My Integer Value = %d" % myintvar)
print("My Float Value = %f" % myfloatvar)
print("My String Value = %s" % mystringvar)
```

#### Initialize/Assign multiple variables simultaneously


```python
a,b = 1,2
print("Value of a = %d" % a)
print("Value of b = %d" % b)

```

    

### List[]


```python
myintlist = []

myintlist.append(1)
myintlist.append(2)

print(myintlist[0])
print(myintlist[1])

mymixlist = [1, 2.5, "Hi"]
print(mymixlist)


```
    

## If-Else


```python
x = 8
remainder = x % 2
if remainder == 0:
    print("x is Even Number")
else:
    print("x is Odd Number")
```

    x is Even Number
    

## If-Elif-Else


```python
x = 3
if  x == 1:
    print("Value of x is 1")
elif x == 2:
    print("Value of x is 2")
elif x == 3:
    print("Value of x is 3")
else:
    print("Value of x is Wrong")
```

    Value of x is 3
    

## If in List


```python
name = "John"
if name in ["John", "David"]:
    print("Your name is either John or David")
```

    Your name is either John or David
    

## For loop in List


```python
mylist = [1, 2, 3, 4]
for x in mylist:
    print(x)
```

    1
    2
    3
    4
    

## Range()


```python
r = range(5)
x = list(r)
print(x)
```

    [0, 1, 2, 3, 4]
    

## For loop & Range


```python
for x in range(5):
    print(x)
```

    0
    1
    2
    3
    4
    

## While loop


```python
i = 0
while i < 5:
    print(i)
    i = i+1
```

    0
    1
    2
    3
    4
    

## break statement


```python
for x in range(5):
    if x == 3:
        break           # It will come out from loop
    print(x)
```

    0
    1
    2
    

## Continue statement


```python
for y in range(5):
    if y == 3:        
        continue       # It will skip the particular iteration/block & return to for
    print(y)
```

    0
    1
    2
    4
    

##  Pass statement


```python
for z in range(5):
    if z == 3:            
        pass           # It will bypass the condition 
    print(z)
```

    0
    1
    2
    3
    4
    

## Function

### Function without parameters


```python
def mymethod1():
    print("Hello Method")

mymethod1()
```

    Hello Method
    

### Function with parameters


```python
def mymethod2(a,b):
    sum = a+b
    print(sum)
    
mymethod2(1,2)
```

    3
    

## Dictionary


```python
contacts = {
    "John" : 6666555544,
    "Jack" : 5555444433,
    "Jill" : 4444333322
}

print(contacts)
```

    {'John': 6666555544, 'Jack': 5555444433, 'Jill': 4444333322}
    

### Iterating through dict 


```python
for name, number in contacts.items():
    print(name, number)
```

    John 6666555544
    Jack 5555444433
    Jill 4444333322
    

### Add to dictionary


```python
contacts["David"] = 9999888871
print(contacts)
```

    {'John': 6666555544, 'Jack': 5555444433, 'Jill': 4444333322, 'David': 9999888871}
    

### Remove from dictionary


```python
del contacts['John']
print(contacts)
```

    {'Jack': 5555444433, 'Jill': 4444333322, 'David': 9999888871}
    

# Importing packages


```python
import math

x = math.sqrt(25)
print(x)
```

    5.0
    
