---
description: Map practice.
---

# 2. Dictionary

nomadCoder 강의 실습 #2

1. main.go

```go
package main

import (
	"fmt"
	"go_tutorial/nomadCoder/2.dictionary/mydict"
)

func main() {
	dictionary := mydict.Dictionary{"first": "First word"}

	// 1. Search Test
	definition, err := dictionary.Search("first")
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println(definition)
	}

	// 2. Add Test
	word := "hello"
	definition2 := "Greeting"
	err2 := dictionary.Add(word, definition2)
	if err2 != nil {
		fmt.Println(err2)
	}
	hello, _ := dictionary.Search(word)
	fmt.Println(hello)

	err3 := dictionary.Add(word, definition2)
	if err3 != nil {
		fmt.Println(err3)
	}
	hello2, _ := dictionary.Search(word)
	fmt.Println(hello2)

	// 3. Update Test
	errUpdate := dictionary.Update(word, "New")
	if errUpdate != nil {
		fmt.Println(errUpdate)
	}
	newWord, _ := dictionary.Search(word)
	fmt.Println(newWord)

	// 4. Delete Test
	errDelete := dictionary.Delete(word)
	if errDelete != nil {
		fmt.Println(errDelete)
	}
	temp, err4 := dictionary.Search(word)
	if err4 != nil {
		fmt.Println(err4)
	} else {
		fmt.Println(temp)
	}

	fmt.Println(dictionary)
}

```



Map은 receiver에 \*를 붙여주지 않아도 메모리에 접근할 수 있다. (자동으로 \*를 붙여준다.)

2\. dict.go

```go
package mydict

import "errors"

// Dictionary type
type Dictionary map[string]string

var (
	errNotFound   = errors.New("Not found")
	errCantUpdate = errors.New("Cant update non-existing word")
	errWordExists = errors.New("That word already exists")
)

// Search for a word
func (d Dictionary) Search(word string) (string, error) {
	value, exists := d[word]

	if exists {
		return value, nil
	}
	return "", errNotFound
}

// Add a word to the dictionary
func (d Dictionary) Add(word string, def string) error {
	_, err := d.Search(word)

	switch err {
	case errNotFound:
		d[word] = def
	case nil:
		return errWordExists
	}
	return nil
}

// Update a word to the dictionary
func (d Dictionary) Update(word, definition string) error {
	_, err := d.Search(word)

	switch err {
	case nil:
		d[word] = definition
	case errNotFound:
		return errCantUpdate
	}
	return nil
}

// Delete a word
func (d Dictionary) Delete(word string) error {
	_, err := d.Search(word)

	switch err {
	case nil:
		delete(d, word)
	case errNotFound:
		return errNotFound
	}
	return nil
}

```

