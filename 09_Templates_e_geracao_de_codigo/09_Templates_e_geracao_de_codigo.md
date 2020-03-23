# Templates e geração de código

O que é um template ?

A programação genérica no C++ se dá em grande parte através de *templates*. Eles parecem mágicos, pois diferente de um objeto, ele não verifica assinaturas.

```cpp
template<typename C>
struct Runner {
	int run(typename C::value_type v) {
		C c(v);
		return c.sum(v+1);
	}
};
```

Como saber se “C” tem o método sum, sem ter uma interface ?
 Isso é validado em tempo de compilação, pois diferente de linguagens  como Java, a função declarada com template existe somente quando é  instanciada.

Vamos ver isso com exemplos e código rodando !

Tendo o arquivo abaixo

```cpp
int sumNonTemplated(int a, int b) {
	return a+b;
}

template <typename T>
T sumTemplated(T a, T b) {
	return a+b;
}

void run() {
}
```

Vamos copilar e listar os símbolos

```bash
g++ -std=c++14 -c temp_gen.cpp && nm -C temp_gen.o
```

E temos o seguinte resultado:

```
0000000000000000 T sumNonTemplated(int, int)
0000000000000014 T run()
```

O sumTemplated não aparece pois ele não existe! Apenas quando o template for instanciado ele vai aparecer, pois um template é **compile-time programming**.

Vamos verificar se isso está correto fazendo uma pequena modificação no nosso exemplo

```cpp
int sumNonTemplated(int a, int b) {
	return a+b;
}
 
template <typename T>
T sumTemplated(T a, T b) {
	return a+b;
}
 
void run() {
	sumTemplated(1, 2);
	sumTemplated(1.0, 2.0);
	sumTemplated(1.0f, 2.0f);
}
```

E o resultado agora é bem diferente

```
W double sumTemplated<double>(double, double)
W float sumTemplated<float>(float, float)
W int sumTemplated<int>(int, int)
T sumNonTemplated(int, int)
T run()
```

Ao fazer as chamadas usando int, double e float, o compilador criou uma **versão** de cada função para cada tipo, e assim, tendo versões concretas, elas  aparecem na listagem de símbolos. Além disso, os tipos das funções  template foram inferidos pelo compilador.

O funcionamento é o mesmo para classes com e sem templates, exceto  pelo fato de que as classes que nunca são usadas também não aparecem na  listagem de símbolos.

```cpp
class NonTemplated {
	int value;
public:
	using value_type = int;
	NonTemplated(int a) : value(a) {}
	int sum(int a) { return value + a; }
};

template <typename T>
class Templated {
	T value;
public:
	using value_type = T;
	Templated(T a) : value(a) {}
	T sum(T a) { return value + a; }
};
//.....
NonTemplated nt{1};
Templated<int> ti{1};
Templated<double> td{1.0};
```

```
W NonTemplated::sum(int)
W NonTemplated::NonTemplated(int)
W NonTemplated::NonTemplated(int)
n NonTemplated::NonTemplated(int)
W Templated<double>::sum(double)
W Templated<double>::Templated(double)
W Templated<double>::Templated(double)
n Templated<double>::Templated(double)
W Templated<int>::sum(int)
W Templated<int>::Templated(int)
W Templated<int>::Templated(int)
n Templated<int>::Templated(int)
```

Temos aqui também uma **versão** para cada tipo gerado.

Ao final, mas não menos importante, como as funções e classes que  usam templates somente existem quando são instanciadas em tempo de  compilação, elas tendem a favorecer a geração de código inline, coisa  muito importante para otimização!

Vamos ver um exemplo, pegando todos os códigos anteriores e compilando com otimização -O2

```bash
g++ -O2 -std=c++14 -c temp_gen.cpp && nm -C temp_gen.o
```

E o resultado

```
T sumNonTemplated(int, int)
T run()
```

O único código gerado foi o sumNonTemplated, que é exportado por padrão. As funções com templates todas ficaram inline.

Eu pensei em falar em template instantiation neste post, mas isso é assunto para um post inteiro.

Até breve !

Código: https://github.com/SimplyCpp/posts/blob/master/09_Templates_e_geracao_de_codigo/temp_gen.cpp
