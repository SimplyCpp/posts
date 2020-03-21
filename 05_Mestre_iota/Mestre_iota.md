# Mestre Iota

[Iota](https://en.wikipedia.org/wiki/Iota) é a nona letra do alfabeto grego, ela é equivalente à letra **i** do nosso alfabeto. Por convenção ou hábito, utilizamos a letra **i** na programação para indicar algum tipo de incrementador, como por exemplo, em um *for-loop*.
```cpp
for (size_t i = 0 /* init */; i < xs.size() /* range end */; ++i /* increment */)
	xs[i] = static_cast<int>(i) + 1;
//xs = 1 2 3 4 5 6 7 8 9 10
```

Curiosamente, **iota**, como identificador, também é utilizado na programação para indicar uma sequência finita e consecutiva de números inteiros, como por exemplo, [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]. Inclusive, originalmente na STL existia a função *iota*, inspirada pela linguagem de programação [APL](https://en.wikipedia.org/wiki/APL_%28programming_language%29), você pode conferir neste link: http://www.sgi.com/tech/stl/iota.html

A função *std::iota* como é percebido foi (re)introduzida no C++ 11. Ela é definida no header **<numeric>**: http://www.cplusplus.com/reference/numeric/iota/. Seu propósito é fornecer uma sequência consecutiva dentro de um intervalo fornecido por um par de *iterators* **[begin, end)** a partir de valor inicial. A sequência consecutiva é gerada através da aplicação de uma função sucessora, que no caso geral é um pré-incremento, onde no C++ isto representa **++i**.

Abaixo alguns exemplos do uso da função *std::iota* com uma diversidade de pares de *iterator* e valores iniciais:
```cpp
std::iota(xs.begin(), xs.end(), 1); //xs = 1 2 3 4 5 6 7 8 9 10

std::iota(xs.begin(), xs.end(), 10); //xs = 10 11 12 13 14 15 16 17 18 19

std::fill(xs.begin(), xs.end(), 0);
std::iota(xs.begin(), xs.begin() + xs.size() / 2, 1); //xs = 1 2 3 4 5 0 0 0 0 0

std::iota(xs.rbegin(), xs.rend(), 0); //xs = 9 8 7 6 5 4 3 2 1 0

std::iota(xs.rbegin(), xs.rend(), 100); //xs = 109 108 107 106 105 104 103 102 101 100
```

Não seria interessante se estas sequências fossem alteradas, ou seja, se conseguíssemos alterar o comportamento da função sucessora de forma determinística ou não? E a resposta é sim! É possível alterarmos o comportamento padrão do *operator++*, uma vez que encapsulamos para um tipo convertível ao tipo do *ForwardIterator* indicado (os dois argumentos iniciais de *std::iota* são *iterators*, assim como a maioria dos algoritmos expostos pela STL), e este tipo tenha implementado o operador de pré-incremento. Algo como o tipo *iota_step* sugerido abaixo:
```cpp
template<typename T> struct iota_step final
{
	explicit iota_step(T init, const T increment) 
		: next(init), step(increment) {}

	iota_step& operator++()
	{
		next += step;
		return *this;
	}

	operator T() const { return next; }

private:
	T next;
	const T step;
};
```

Você poderá utilizar o iota_step juntamente com std::iota da seguinte forma:
```cpp
std::iota(xs.begin(), xs.end(), iota_step<int>(2, 2)); //xs = 2 4 6 8 10 12 14 16 18 20

std::iota(xs.begin(), xs.end(), iota_step<int>(0, -2)); //xs = 0 -2 -4 -6 -8 -10 -12 -14 -16 -18
```

Ou se preferir utilizar uma função para construir o iota_step e se beneficiar da inferência do tipo:
```cpp
std::iota(xs.begin(), xs.end(), make_iota_step(100, 100)); //xs = 100 200 300 400 500 600 700 800 900 1000
```

Comentei acima que poderia fazer isto de forma não determinística, neste caso, que a função sucessora seja imprevisível ou aleatória. Basta seguir as mesmas considerações do iota_step e programar o operator++ com algum tipo de gerador de números pseudoaleatórios, como segue:
```cpp
template<typename T> struct iota_random_with_uniform_distribution final
{
	explicit iota_random_with_uniform_distribution(const T minInclusive, const T maxInclusive, unsigned int seed = 5489U)
		: rnd(std::bind(std::uniform_int_distribution<T>(minInclusive, maxInclusive), std::default_random_engine(seed)))
	{
		next = rnd();
	}

	iota_random_with_uniform_distribution& operator++()
	{
		next = rnd();
		return *this;
	}

	operator T() const { return next; }

private:
	T next;
	std::function<T(void)> rnd;
};
```

Assim você utilizará a característica básica de std::iota através de um comportamento estendido. Isto é divertido ou no mínimo curioso, não acha?

BTW, este post não é uma homenagem ao despertar da força! 

Fontes: 
https://github.com/SimplyCpp/examples/blob/master/understanding_iota.hpp
https://github.com/SimplyCpp/examples/blob/master/understanding_iota_program.cpp
