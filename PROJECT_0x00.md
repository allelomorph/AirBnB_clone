# (263) 0x00. AirBnB clone - The console
Foundations > Higher-level programming > AirBnB clone

---

### Project author
Guillaume Salva

### Assignment dates
06-25-2020 to 07-03-2020

### Description
First of two projects building the first iteration of a website cloning the basic features of the [`airbnb.com` main page, circa 2014-2017](https://web.archive.org/web/20170206112507/https://www.airbnb.com/).

Review of concepts from [`holbertonschool-higher_level_programming`](https://github.com/allelomorph/holbertonschool-higher_level_programming/) like unit testing, JSON serialization/deserialization, and `*args`/`**kwargs` in Python, plus an introduction to UUIDs and the `cmd` module. Creating a command line interpreter to store data objects in local JSON files.

## General requirements

### Python
* Interpreter conditions:
  * Ubuntu 14.04 LTS
  * python3 (version 3.4.3)
* First line of executable scripts will be `#!/usr/bin/python3`
* Compliance with linter:
  * `pep8` (version 1.7.*) (now known as `pycodestyle`)
* Docstrings are expected to follow the [Google style guide](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html):
  * Per module (`python3 -c 'print(__import__("my_module").__doc__)'`)
  * Per class (`python3 -c 'print(__import__("my_module").MyClass.__doc__)'`)
  * Per function
    * both inside a class (`python3 -c 'print(__import__("my_module").MyClass.my_function.__doc__)'`)
    * and outside a class (`python3 -c 'print(__import__("my_module").my_function.__doc__)'`)
* Test scripts will typically not be in same directory as the task solutions, use `export PYTHONPATH='.'` before running test scripts from project directory to allow includes
* Unit tests will be required on some projects:
  * using the [unittest module](https://docs.python.org/3.4/library/unittest.html#module-unittest)
  * located in a `tests/` folder, with a file structure mimicing that of your project, but with a `test_` prefix added to all file/directory names
  * tests should be capable of being run with `python3 -m unittest discover tests`, or individually per file with `python3 -m unittest <test file>`

### `cmd` CLI
Shell should work in interactive mode:
```
$ ./console.py
(hbnb) help

Documented commands (type help <topic>):
========================================
EOF  help  quit

(hbnb) 
(hbnb) 
(hbnb) quit
$
```
But also in non-interactive mode:
```
$ echo "help" | ./console.py
(hbnb)

Documented commands (type help <topic>):
========================================
EOF  help  quit
(hbnb) 
$
$ cat test_help
help
$
$ cat test_help | ./console.py
(hbnb)

Documented commands (type help <topic>):
========================================
EOF  help  quit
(hbnb) 
$
```

---

## Mandatory Tasks

### :white_large_square: 0. README, AUTHORS
* Write a `README.md`:
    * description of the project
    * description of the command interpreter:
        * how to start it
        * how to use it
        * examples
* You should have an `AUTHORS` file at the root of your repository, listing all individuals having contributed content to the repository. For the format, reference [Docker’s AUTHORS page](https://github.com/moby/moby/blob/master/AUTHORS)
* You should use branches and pull requests on GitHub - it will help you as team to organize your work

File(s): [`README.md`](./README.md) [`AUTHORS`](./AUTHORS)

### :white_check_mark: 1. Be Pycodestyle compliant!
Write beautiful code that passes the pycodestyle checks.

File(s): all Python files (`*.py`)

### :white_check_mark: 2. Unittests
All your files, classes, functions must be tested with unit tests
```
$ python3 -m unittest discover tests
...
OK
$
```
Unit tests must also pass in non-interactive mode:
```
$ echo "python3 -m unittest discover tests" | bash
...
OK
$
```

File(s): [`tests/`](./tests/)

### :white_check_mark: 3. BaseModel
Write a class `BaseModel` that defines all common attributes/methods for other classes:
* `models/base_model.py`
* Public instance attributes:
    * `id`: string - assign with an `uuid` when an instance is created:
        * you can use `uuid.uuid4()` to generate unique id but don’t forget to convert to a string
        * the goal is to have unique `id` for each `BaseModel`
    * `created_at`: datetime - assign with the current datetime when an instance is created
    * `updated_at`: datetime - assign with the current datetime when an instance is created and it will be updated every time you change your object
* `__str__`: should print: `[<class name>] (<self.id>) <self.__dict__>`
* Public instance methods:
    * `save(self)`: updates the public instance attribute `updated_at` with the current datetime
    * `to_dict(self)`: returns a dictionary containing all keys/values of `__dict__` of the instance:
        * by using `self.__dict__`, only instance attributes set will be returned
        * a key `__class__` must be added to this dictionary with the class name of the object
        * `created_at` and `updated_at` must be converted to string object in ISO format:
            * format: `%Y-%m-%dT%H:%M:%S.%f` (ex: `2017-06-14T22:31:03.285259`)
            * you can use `isoformat()` of `datetime` object
        * This method will be the first piece of the serialization/deserialization process: create a dictionary representation with “simple object type” of our `BaseModel`

File(s): [`models/base_model.py`](./models/base_model.py) [`models/__init__.py`](./models/__init__.py) [`tests/`](./tests/)

### :white_check_mark: 4. Create BaseModel from dictionary
Previously we created a method to generate a dictionary representation of an instance (method to_dict()).

Now it’s time to re-create an instance with this dictionary representation.
```
<class 'BaseModel'> -> to_dict() -> <class 'dict'> -> <class 'BaseModel'>
```
Update `models/base_model.py`:
* `__init__(self, *args, **kwargs)`:
    * you will use `*args, **kwargs` arguments for the constructor of a `BaseModel`
    * `*args` won’t be used
    * if `kwargs` is not empty:
        * each key of this dictionary is an attribute name (`__class__` from `kwargs` is the only one that should not be added as an attribute)
        * each value of this dictionary is the value of this attribute name
        * **Warning**: `created_at` and `updated_at` are strings in this dictionary, but inside your `BaseModel` instance is working with `datetime` object. You have to convert these strings into `datetime` object
    * otherwise:
        * create `id` and `created_at` as you did previously (new instance)

File(s): [`models/base_model.py`](./models/base_model.py) [`tests/`](./tests/)

### :white_check_mark: 5. Store first object
Now we can recreate one `BaseModel` from another by using a dictionary representation:
```
<class 'BaseModel'> -> to_dict() -> <class 'dict'> -> <class 'BaseModel'>
```
But this is not persistent: every time you launch the program, you don’t restore all objects created before. So the first means of storage we will implement is to save these objects to a file.

Writing the dictionary representation to a file won’t be relevant:
* Python doesn’t know how to convert a string to a dictionary (easily)
* It’s not human readable
* Using this file with another program in Python or other language will be hard

So, you will convert the dictionary representation to a JSON string. JSON is a standard representation of a data structure. This format allows for both human readability and parsing by other programming languages.

Now the flow of serialization-deserialization will be:
```
<class 'BaseModel'> -> to_dict() -> <class 'dict'> -> JSON dump -> <class 'str'> -> FILE -> <class 'str'> -> JSON load -> <class 'dict'> -> <class 'BaseModel'>
```

Be careful to distinguish the Python `__str__` or `__repr__` of an object, eg. `{ '12': { 'numbers': [1, 2, 3], 'name': "John" } }` string from the JSON format serialization: `'{ "12": { "numbers": [1, 2, 3], "name": "John" } }'`.

Write a class `FileStorage` that serializes instances to a JSON file and deserializes JSON file to instances:
* `models/engine/file_storage.py`
* Private class attributes:
    * `__file_path`: string - path to the JSON file (ex: `file.json`)
    * `__objects`: dictionary - empty but will store all objects by `<class name>.id` (ex: to store a `BaseModel` object with `id=12121212`, the key will be `BaseModel.12121212`)
* Public instance methods:
    * `all(self)`: returns the dictionary `__objects`
    * `new(self, obj)`: sets in `__objects` the `obj` with key `<obj class name>.id`
    * `save(self)`: serializes `__objects` to the JSON file (path: `__file_path`)
    * `reload(self)`: deserializes the JSON file to `__objects` (only if the JSON file (`__file_path`) exists; otherwise, do nothing. If the file doesn’t exist, no exception should be raised)

Update `models/__init__.py`: to create a unique `FileStorage` instance for your application
* import `file_storage.py`
* create the variable `storage`, an instance of `FileStorage`
* call `reload()` method on this variable

Update `models/base_model.py`: to link your `BaseModel` to `FileStorage` by using the variable `storage`
* import the variable `storage`
* in the method `save(self)`:
    * call `save(self)` method of `storage`
* `__init__(self, *args, **kwargs)`:
    * if it’s a new instance (not from a dictionary representation), add a call to the method `new(self)` on `storage`

File(s): [`models/engine/file_storage.py`](./models/engine/file_storage.py) [`models/engine/__init__.py`](./models/engine/__init__.py) [`models/__init__.py`](./models/__init__.py) [`models/base_model.py`](./models/base_model.py) [`tests/`](./tests/)

### :white_check_mark: 6. Console 0.0.1
Write a program called `console.py` that contains the entry point of the command interpreter:
* You must use the module `cmd`
* Your class definition must be: `class HBNBCommand(cmd.Cmd):`
* Your command interpreter should implement:
    * `quit` and `EOF` to exit the program
    * `help` (this action is provided by default by `cmd` but you should keep it updated and documented as you work through tasks)
    * a custom prompt: `(hbnb)`
    * an empty line + ENTER shouldn’t execute anything
* Your code should not be executed when imported

File should end with:
```python
if __name__ == '__main__':
    HBNBCommand().cmdloop()
```
to make your program executable except when imported.
```
~/AirBnB$ ./console.py
(hbnb) help

Documented commands (type help <topic>):
========================================
EOF  help  quit

(hbnb) 
(hbnb) help quit
Quit command to exit the program

(hbnb) 
(hbnb) 
(hbnb) quit 
~/AirBnB$ 
```

File(s): [`console.py`](./console.py)

### :white_check_mark: 7. Console 0.1
Update your command interpreter (`console.py`) to have these commands:
* `create`: Creates a new instance of `BaseModel`, saves it (to the JSON file) and prints the `id`. Ex: `create BaseModel`
    * If the class name is missing, print `** class name missing **` (ex: `create`)
    * If the class name doesn’t exist, print `** class doesn't exist **` (ex: `create MyModel`)
* `show`: Prints the string representation of an instance based on the class name and `id`. Ex: `show BaseModel 1234-1234-1234`
    * If the class name is missing, print `** class name missing **` (ex: `show`)
    * If the class name doesn’t exist, print `** class doesn't exist **` (ex: `show MyModel`)
    * If the `id` is missing, print `** instance id missing **` (ex: `show BaseModel`)
    * If the instance of the class name doesn’t exist for the `id`, print `** no instance found **` (ex: `show BaseModel 121212`)
* `destroy`: Deletes an instance based on the class name and `id` (save the change into the JSON file). Ex: `destroy BaseModel 1234-1234-1234`
    * If the class name is missing, print `** class name missing **` (ex: `destroy`)
    * If the class name doesn’t exist, print `** class doesn't exist **` (ex: `destroy MyModel`)
    * If the `id` is missing, print `** instance id missing **` (ex: `destroy BaseModel`)
    * If the instance of the class name doesn’t exist for the `id`, print `** no instance found **` (ex: `destroy BaseModel 121212`)
* `all`: Prints all string representation of all instances based or not on the class name. (ex: `all BaseModel` or `all`
    * The printed result must be a list of strings
    * If the class name doesn’t exist, print `** class doesn't exist **` (ex: `all MyModel`)
* `update`: Updates an instance based on the class name and `id` by adding or updating attribute (save the change into the JSON file) (ex: `update BaseModel 1234-1234-1234 email "aibnb@mail.com"`)
    * Usage: `update <class name> <id> <attribute name> "<attribute value>"`
    * Only one attribute can be updated at the time
    * You can assume the attribute name is valid (exists for this model)
    * The attribute value must be casted to the attribute type
    * If the class name is missing, print `** class name missing **` (ex: `update`)
    * If the class name doesn’t exist, print `** class doesn't exist **` (ex: `update MyModel`)
    * If the `id` is missing, print `** instance id missing **` (ex: `update BaseModel`)
    * If the instance of the class name doesn’t exist for the `id`, print `** no instance found **` (ex: `update BaseModel 121212`)
    * If the attribute name is missing, print `** attribute name missing **` (ex: `update BaseModel existing-id`)
    * If the value for the attribute name doesn’t exist, print `** value missing **` (ex: `update BaseModel existing-id first_name`)
    * All other arguments should not be used (ex: `update BaseModel 1234-1234-1234 email "aibnb@mail.com" first_name "Betty"` = `update BaseModel 1234-1234-1234 email "aibnb@mail.com"`)
    * `id`, `created_at` and `updated_at` can’t be updated. You can assume they won’t be passed in the update command
    * Only “simple” arguments can be updated: string, integer and float. You can assume nobody will try to update list of ids or datetime

Let’s add some rules:
* You can assume arguments are always in the right order
* Each arguments are separated by a space
* A string argument with a space must be between double quote
* The error management starts from the first argument to the last one

File(s): [`console.py`](./console.py)

### :white_check_mark: 8. First User
Write a class `User` that inherits from `BaseModel`:
* `models/user.py`
* Public class attributes:
    * `email`: string - empty string
    * `password`: string - empty string
    * `first_name`: string - empty string
    * `last_name`: string - empty string

Update `FileStorage` to manage correctly serialization and deserialization of `User`.

Update your command interpreter (`console.py`) to allow `show`, `create`, `destroy`, `update` and `all` used with `User`.

File(s): [`models/user.py`](./models/user.py) [`models/engine/file_storage.py`](./models/engine/file_storage.py) [`console.py`](./console.py) [`tests/`](./tests/)

### :white_check_mark: 9. More classes!
Write all those classes that inherit from `BaseModel`:
* `State` (`models/state.py`):
    * Public class attributes:
        * `name`: string - empty string
* `City` (`models/city.py`):
    * Public class attributes:
        * `state_id`: string - empty string: it will be the State.id
        * `name`: string - empty string
* `Amenity` (`models/amenity.py`):
    * Public class attributes:
        * `name`: string - empty string
* `Place` (`models/place.py`):
    * Public class attributes:
        * `city_id`: string - empty string: it will be the `City.id`
        * `user_id`: string - empty string: it will be the `User.id`
        * `name`: string - empty string
        * `description`: string - empty string
        * `number_rooms`: integer - 0
        * `number_bathrooms`: integer - 0
        * `max_guest`: integer - 0
        * `price_by_night`: integer - 0
        * `latitude`: float - 0.0
        * `longitude`: float - 0.0
        * `amenity_ids`: list of string - empty list: it will be the list of `Amenity.id` later
* `Review` (`models/review.py`):
    * Public class attributes:
        * `place_id`: string - empty string: it will be the `Place.id`
        * `user_id`: string - empty string: it will be the `User.id`
        * `text`: string - empty string

File(s): [`models/state.py`](./models/state.py) [`models/city.py`](./models/city.py) [`models/amenity.py`](./models/amenity.py) [`models/place.py`](./models/place.py) [`models/review.py`](./models/review.py) [`tests/`](./tests/)

### :white_check_mark: 10. Console 1.0
Update `FileStorage` to manage correctly serialization and deserialization of all our new classes: `Place`, `State`, `City`, `Amenity` and `Review`.

Update your command interpreter (`console.py`) to allow those actions: `show`, `create`, `destroy`, `update` and `all` with all classes created previously.

Enjoy your first console!

File(s): [`console.py`](./console.py) [`models/engine/file_storage.py`](./models/engine/file_storage.py) [`tests/`](./tests/)

## Advanced Tasks

### :white_check_mark: 11. All instances by class name
Update your command interpreter (`console.py`) to retrieve all instances of a class by using: `<class name>.all()`.

File(s): [`console.py`](./console.py)

### :white_check_mark: 12. Count instances
Update your command interpreter (`console.py`) to retrieve the number of instances of a class: `<class name>.count()`.

File(s): [`console.py`](./console.py)

### :white_check_mark: 13. Show
Update your command interpreter (`console.py`) to retrieve an instance based on its `id`: `<class name>.show(<id>)`.

Error management must be the same as previously.

File(s): [`console.py`](./console.py)

### :white_check_mark: 14. Destroy
Update your command interpreter (`console.py`) to destroy an instance based on its `id`: `<class name>.destroy(<id>)`.

Error management must be the same as previously.

File(s): [`console.py`](./console.py)

### :white_check_mark: 15. Update
Update your command interpreter (`console.py`) to update an instance based on its `id`: `<class name>.update(<id>, <attribute name>, <attribute value>)`.

Error management must be the same as previously.

File(s): [`console.py`](./console.py)

### :white_large_square: 16. Update from dictionary
Update your command interpreter (`console.py`) to update an instance based on its `id` with a dictionary: `<class name>.update(<id>, <dictionary representation>)`.

Error management must be the same as previously.

File(s): [`console.py`](./console.py)

### :white_large_square: 17. Unittests for the Console!
Write unittests for all features of `console.py`!

For testing the console, you should “intercept” its `stdout`, we highly recommend you to use:
```python
with patch('sys.stdout', new=StringIO()) as f:
    HBNBCommand().onecmd("help show")
```
Otherwise, you will have to re-write the console by replacing `precmd` by default.

File(s): [`tests/test_console.py`](./tests/test_console.py)

---

## Student team
* **Samuel Pomeroy** - [allelomorph](github.com/allelomorph)
* **Nic Basilio** - [bnbasilio](github.com/bnbasilio)
