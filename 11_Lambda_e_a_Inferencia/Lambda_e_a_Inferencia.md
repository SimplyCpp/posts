# Lambda e a Inferência

Duas das coisas que mais me agradaram no C++ atual são: a **inferência de tipo** e as **expressões lambda** (ou simplesmente *lambda*).

Neste post, quero focar em 5 pontos interessantes sobre estes assuntos. São eles:

1. O que é um lambda?
1. Lambda e Functor, qual a relação entre eles?
1. Como funciona a inferência feita em um lambda?
1. Lista de captura de um lambda
1. Legibilidade interessante

## O que é um *lambda*?

Muito resumidamente, o cálculo *lambda* ([lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus)) é uma representação computacional de uma função matemática.

```apl
Ex λ: (x, y) -> 2 * x + y
```

No C++, o *lambda* é uma função com lista de captura, parâmetros e um retorno (que pode ser void).

## *Lambda* e *Functor*, qual a relação entre eles?

A implementação de um *lambda* é a mesma implementação de um *functor*! Ou seja, o compilador mapeia/transforma um *lambda* em um *functor* equivalente.

* O *lambda* tem: lista de captura, função recebendo parâmetros e um retorno
* O *functor* tem: constructor, operador () recebendo parâmetros e um retorno
* O *lambda* é uma maneira interessante para definir um *functor* anônimo.

Os dois vão virar código *inline*, mas qual tem a sintaxe mais amigável?

Vamos comparar as duas implementações de uma função que multiplica um valor por outro pré-definido.

*Functor*:

```cpp
template<typename T>
class F {
	T &_b;
public:
	F(T &b) : _b(b) { }
	auto operator()(T v) {
		return v * _b;		
	}
};
```

*Lambda*:

```cpp
[&b](auto v) {
	return v*b;
};
```

Note que a **referência** da variável b será passado junto com a função lambda.

## Como funciona a inferência feita em um *lambda*?

Agora, vamos focar somente no *lambda*.

O que está sendo inferido?

1. Parâmetro v
1. Retorno, se tem o qual é
1. O tipo do *lambda* baseado em **std::function**

```cpp
//Super duper automatic
auto f1 = [&b](auto v) {
	return v*b;
};
```

* Parâmetro v
* Retorno: resultado do operador * dos tipos de v e b
* O tipo do lambda baseado em **std::function**

```cpp
//More restrict
auto f2 = [&b](auto v) -> decltype(v*b) {
	return v*b;
};
```

* O tipo do lambda, sendo **std::function<int(int)>**

```cpp
//very restrict
auto f3 = [&b](int v) -> int {
	return v*b;
};
```

## Lista de captura de um lambda

O *lambda* tem alguns tipos de captura:

```
[a] -> captura a variável a por valor
[=] -> captura todas as variáveis por valor
[&a] -> captura a variável a por referência
[&] -> captura todas as variáveis por referência
[a, &b] -> captura a por valor e b por referência
[] -> não captura nada
```

## Legibilidade interessante

E veja se não fica muito boa a legibilidade.

```cpp
fix_parser parser;

msg_transport::received_msg(transport, [&m](auto &data) {
	message m = parser.get_msg(data);
	processor.new_msg(m);
});

msg_transport::received_cmd(transport, [&m](auto &data) {
	command c = parser.get_msg(data);
	admin.new_msg(c);
});
```

Se esses códigos fossem mostrados para um programador C++ de uns 10  anos atrás, ele certamente não acreditaria que é a mesma linguagem!

Código:

* https://github.com/SimplyCpp/posts/blob/master/11_Lambda_e_a_Inferencia/lambda.cpp
* https://github.com/SimplyCpp/posts/blob/master/11_Lambda_e_a_Inferencia/lambda2.cpp
