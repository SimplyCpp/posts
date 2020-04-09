# Legibilidade com C++14 – get_numeric

Eu escrevi um código em C++ há um tempo atrás e quando olhei hoje vi como a legibilidade mudou desde o C++11.

Neste código eu vi algumas coisas interessantes. Ele serve bem de exemplo para alguns tipos de codificação

1. Codificação concreta
2. Codificação genérica
3. Codificação concreta com C++ moderno
4. Codificação genérica com C++ moderno

Vamos ver agora cada um deles.



## O algoritmo

O algoritmo que eu implementei é bem simples. Ele itera em uma *std::string* de entrada e copia somente os caracteres não numéricos para a *string* de saída.
 Ex: “12asew3d45ddf678ee9 0” => “1234567890”

### Codificação concreta

Essa é a versão mais simples de todas, de implementar e de entender.

```cpp
struct numerical_appender {
	numerical_appender(std::string &buf) : _buf(buf) { }
	void operator()(const char c) {
		if( c >= '0' && c <= '9' ) _buf.push_back(c);
	}
private:
	std::string &_buf;
};

void get_numeric(const std::string &input, std::string &output) 
{
	numerical_appender appender(output);
	std::for_each(input.begin(), input.end(), appender);
}
```

Nós estamos usando o *for_each* com um *Functor numerical_appender* para que ele adicione na *string* de saída somente os caracteres numéricos.

É uma implementação trivial. Para cada caractere da entrada, o *operator()* será chamado.

### Codificação genérica

Aqui nós temos uma diferença básica. Ao invés de receber uma *string*, o algoritmo trabalha com iteradores somente e dessa forma suporta qualquer tipo de vetor, podendo ser uma *string*, *vector*, *list*, etc…

```cpp
template<class OutputIterator, class ValueType>
struct numerical_appender {
	numerical_appender(OutputIterator it) : _it(it) { }
	void operator()(const ValueType c) {
		if( c >= '0' && c <= '9' ) {
			*_it = c;
			_it++;
		}
	}
private:
	OutputIterator _it;
};

template<class T>
void get_numeric(const T &input, T &output) 
{
	typedef std::back_insert_iterator<T> it_type;
	it_type it = std::back_inserter(output);
	numerical_appender<it_type, typename T::value_type> appender(it);
	std::for_each(input.begin(), input.end(), appender);
}
```

### Codificação concreta com C++ moderno

C++ 11 e o *lambda* vieram facilitar a nossa vida e melhorar a legibilidade realmente.
Vamos fazer a implementação concreta da nossa função:

```cpp
std::string get_numeric(const std::string &input) 
{
	std::string output;
	std::for_each(begin(input), end(input), [&output](const char c) {
		if( c >= '0' && c <= '9' ) output.push_back(c);
	});
	return output;
}
```

Bem mais simples!

Temos agora uma função encadeada dentro de *get_numeric* e fazendo o filtro.

### Codificação genérica com C++ moderno

Agora temos uma surpresa interessante. Vejamos a diferença dela para a versão concreta.

```cpp
template<typename T>
T get_numeric(const T &input) 
{
	T output;
	std::for_each(begin(input), end(input), [&output](auto c) {
		if( c >= '0' && c <= '9' ) output.push_back(c);
	});
	return output;
}
```

Note que ela é quase igual à versão concreta. Temos o *template* de diferença e o *auto* no *lambda*. Fácil, não acha?

### C++ 14 – STL

No C++ 14 (desde o C++ 11, na verdade) temos um método novo na STL chamado **copy_if**. Esse método faz basicamente o que o *std::copy* faz, porém tendo um predicado para definir se o caractere será copiado.

```cpp
template<typename T>
T get_numeric(const T &input) 
{
	T output;
	std::copy_if(begin(input), end(input), std::back_inserter(output), [](auto c) {
		return ( c >= '0' && c <= '9' );
	});
	return output;
}
```

Simples, não?

Uma outra forma de implementar esse mesmo método seria usar o iterador com predicado. Fica para quem tiver curiosidade:

[Iterator com predicado - Simply C++](https://github.com/SimplyCpp/posts/blob/master/04_Iterator_com_predicado/Iterator_com_predicado.md)

### Referências

#### Fonte:

- [numeric.cpp](https://github.com/SimplyCpp/posts/blob/master/16_Legibilidade_com_Cpp14-get_numeric/numeric.cpp)

#### Referências:

- [cplusplus.com - copy_if](http://www.cplusplus.com/reference/algorithm/copy_if/)
- [cplusplus.com - for_each](http://www.cplusplus.com/reference/algorithm/for_each/)
