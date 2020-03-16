# Olá, mundo!

```cpp
#include <string>
#include <array>
#include <algorithm>
#include <iostream>

int main(int argc, char *argv[]) {

	/*
	 * I don’t want to make the choice between elegant, 
	 * which Simula was for this problem,
	 * and efficient, which BCPL was. I want both
	 * (Bjarne Stroustrup, 2015-02-05)
	 */
        
	std::array<std::string, 2> authors { "Fabio", "Thiago" };
	
	std::for_each(std::begin(authors), std::end(authors), 
		[](const std::string &author) { 
			std::cout << "Hello from " << author << "  !\n" ; 
		});
		
	for(const auto &author : authors) {
		std::cout << author << " hopes you like the site !\n" ; 
	}
 
 	return 0;

}
```

**Update:**

Me perguntaram a razão de ter feito esse floreamento todo no código, pois não é eficiente.
Então, eu achei interessante colocar aqui como seria isso em um código real.

Se eu estivesse escrevendo o código em C

```cpp
printf("Hello from Fabio !\n");
printf("Hello from Thiago !\n");
printf("Fabio hopes you like the site !\n");
printf("Thiago hopes you like the site !\n");
```

Se isso fosse um programa real em C++ (supondo que você encontrou uma  boa razão para escrever um programa de hello world profissionalmente).

```cpp
std::cout << "Hello from Fabio !" << std::endl;
std::cout << "Hello from Thiago !" << std::endl;
std::cout << "Fabio hopes you like the site !" << std::endl;
std::cout << "Thiago hopes you like the site !" << std::endl;
```

Até mais !
Feedbacks são sempre bem vindos !

Fonte: https://github.com/SimplyCpp/posts/blob/master/01_ola-mundo/hello.cpp