# Range loops – Escrevendo código seguro

Eu estava olhando um código de um sistema e me deparei com um trecho  que me fez torcer o nariz. O código funcionava, mas imediatamente vi  dois problemas potenciais:


1. Bug de int/unsigned int
	* Um vector::size() – 1
	* Se size for 0, o resultado será 4294967295 ou 0xFFFFFFFF!
1. Código confuso e facilmente **quebrável**
	* msgs[count] – Não tem offset dinâmico
	* Um count inválido pode quebrar o programa



```cpp
//Bad code. Don't try it at home.
int count = 1;
while (count < msgs.size() - 1)
{
	const Message &item = msgs[count];
	PublishMessage(item);
	count++;
}
```

Eu suponho que a pessoa que desenvolveu o código fez dessa forma para pegar os itens que estavam no *vector*, excluindo os itens da borda (0:n), mas como seria essa implementação da **forma STL de se programar** ?

## Três possibilidades

```cpp
//First possibility
for(auto it = msgs.cbegin(); it != msgs.cend(); it++) {
	PublishMessage(*it);
}

//Second possibility
for(const auto &msg : msgs) {
	PublishMessage(msg);
}

//Thrird possibility
for_each(begin(msgs), end(msgs), [](const auto &msg) {
	PublishMessage(msg);
});
```

Todas elas tem um problema com relação à implementação original. A implementação original pega o *range* de [1:size-1). Estas formas que eu apresentei pegam o *range* de [0:size). 

Como fazer então ?

A forma mais **inocente e com os mesmos problemas** da implementação que eu vi seria:

```cpp
//Also bad code. Don't try at home.
for(auto it = msgs.cbegin()+1; it != msgs.cend()-1; it++) {
	PublishMessage(*it);
}
```

Se o tamanho do *vector* for 0, *msgs.cbegin() == msgs.cend()*, e teremos um problema, pois *msgs.cbegin()+1* estaria apontando para uma região da memória mais adiante da apontada por *msgs.cend()-1*. Vemos também que os outros exemplos de *ranged-for* não nos dão a possibilidade de mudar o intervalo com *offsets*. Qual seria a solução então ?

A definição de um *container* no qual é possível executar iterações, (grosseiramente) é um objeto que possui as funções *begin* e *end*. Então, fiz uma solução interessante para o caso, algo parecido ou no estilo de uma *view*.

## Solução adequada – Range!

Para começar, precisamos de uma estrutura que implemente um *range*

```cpp
template<typename T>
struct range {
	range(const T &cont) : _b(cont.begin()), _e(cont.end()) {}
	range(typename T::const_iterator b, typename T::const_iterator e) : _b(b), _e(e) {}

	typename T::const_iterator &begin() { return _b; }
	typename T::const_iterator &end() { return _e; }

private:
	typename T::const_iterator _b, _e;
};
```

Agora já temos a nossa estrutura contendo *begin* e *end* e dois iteradores. Só falta criarmos uma função que aponte os iteradores para um *container* definindo o *range* [1:size-1)

```cpp
//Positive end means [start:start+end)
//Negative end means [start:size-end)
template<typename T>
range<T> narrow_range(const T &cont, int start, int end) {

	int items = std::distance(cont.cbegin(), cont.cend());
	if( items == 0 || end == 0 ) //Checking for empty range
		return range<T>(cont.cend(), cont.cend());
  
	/*
	    Some other checkings stripped to make it less verbose
	*/
	
	if( end > 0 ) {
		return range<T>(cont.cbegin()+start, cont.cbegin()+start+end);
	} else 
		return range<T>(cont.cbegin()+start, cont.cend()+end);
}
```

Temos agora uma função que retorna de forma segura o *range* [1:size-1) mesmo quando o vetor é vazio ou contém menos de dois elementos.

Segue abaixo um exemplo de uso da função *narrow_range*

```cpp
template<typename T>
void print1(T &msgs) {
	for(const auto &msg : msgs) {
		PublishMessage(msg);
	}
}

template<typename T>
void print2(T &msgs) {
	for_each(begin(msgs), end(msgs), [](const auto &msg) {
		PublishMessage(msg);
	});
}
//--------------

vector<Message> msgs {"MSG1","MSG2","MSG3","MSG4","MSG5"};
vector<Message> empty_msgs {};

auto msgs_sub1 = narrow_range(msgs, 1, -1); //[1:-1]
auto msgs_sub2 = narrow_range(msgs, 1, 2);  //[1:2]

auto msgs_sub3 = narrow_range(msgs, 1, 5);  //Empty
auto msgs_sub4 = narrow_range(msgs, 1, -5);  //Empty
auto msgs_sub5 = narrow_range(empty_msgs, 1, -1);  //Empty

print1(msgs_sub1); //shows items 1, 2, 3
print2(msgs_sub1); //shows items 1, 2, 3
print2(msgs_sub2); //shows items 1, 2

print2(msgs_sub3); //empty
print2(msgs_sub4); //empty
print2(msgs_sub5); //empty
```

Fontes:

* https://github.com/SimplyCpp/posts/blob/master/13_Range_loops_Escrevendo_codigo_seguro/loop1.cpp
* https://github.com/SimplyCpp/posts/blob/master/13_Range_loops_Escrevendo_codigo_seguro/loop2.cpp