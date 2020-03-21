# Iterator com predicado, o que é isso?

O *iterator* é um objeto que aponta ou indica um elemento em uma extensão de elementos, tais como containers da STL (por exemplo: *std::vector*) ou um *array*.

A **Standard Template Library (STL)** possui uma boa variedade de *containers* ou coleções que podem armazenar elementos, onde uma das operações naturais sobre estes *containers* são as iterações – assim é possível percorrer ou visitar cada elemento presente. Certa vez, li uma definição do relacionamento entre os principais recursos da STL (*iterators*, *algorithms* e *containers*) que era algo do tipo: **algorithms** atuam em **containers** através de **iterators**. Esta é uma definição simplificada e válida do que chamamos de “STL-way”. Se tentar copiar elementos de um *std::vector* para outro *std::vector*, provavelmente utilizará um *std::copy* e passará um par de *iterators* (*begin* e *end*) para indicar a extensão a ser copiada, além de utilizar um outro *iterator* para indicar o destino:
```cpp
std::vector<int> xs = { 1, 2, 3, 4, 5, 6, 7, 8 }, ys;
ys.resize(xs.size());
std::copy(xs.begin(), xs.end(), ys.begin());
```

A idéia básica do *iterator* vêm dos ponteiros, afinal um ponteiro é um tipo de *iterator* rústico, onde as operações triviais são incremento (ir para o próximo elemento quando o incremento positivo, operator++), dereferencia (operator*) e outras operações aritméticas. O *iterator* no C++ amplia a capacidade dos ponteiros de memória introduzindo certas categorias que determinam o que é permitido, em termos de operações, se fazer com aquele *iterator* – neste post iremos apresentar um *iterator* do tipo *Forward*, em futuros posts apresentaremos cada uma das categorias existentes, ok?

Quando utilizamos um par de *iterators* de um *container* qualquer (normalmente *begin* e *end*, mas também pode ser algo parcial: *begin()* e *begin() + 4* dependendo do tipo), isto significa que a intenção é de iterar os elementos deste *container* um a um, até chegarmos ao final. Isto é feito declarativamente da seguinte forma, utilizando o algoritmo *std::replace* como exemplo:
```cpp
std::vector<int> xs = { 1, 2, 3, 4, 5, 6, 7, 8 };
std::replace(xs.begin(), xs.end(), 5, 500); //1, 2, 3, 4, 500, 6, 7, 8
```
Ou em sua forma imperativa:
```cpp
auto first = xs.begin();
auto last = xs.end();
while (first != last) 
{
    if (*first == 5) 
        *first = 500;
    ++first;
}
//xs = 1, 2, 3, 4, 500, 6, 7, 8
```

No entanto, mesmo visitando cada elemento do intervalo aberto (note que usando uma notação matemática, o intervalo indicado pelo par é [*begin*, *end*) é fechado no inicio e aberto no final – por isso também que o *end* é conhecido como *past-end*, ou seja, o próximo elemento do último elemento desejado), não queremos visitar ou alterar todos eles, simplesmente desejamos descartar ou filtrar alguns deles. Para isto, existem algoritmos, normalmente sufixados com **_if** que possuem um predicado para filtragem, por exemplo, *std::copy_if* para copiar apenas elementos com uma certa caracteristica:
```cpp
std::vector<int> xs = { 1, 2, 3, 4, 5, 6, 7, 8 }, ys;
ys.resize(xs.size() / 2);
	
auto is_odd = [](int i) {
    return (i & 0x1) == 0x1;
};

std::copy_if(xs.begin(), xs.end(), ys.begin(), is_odd); //ys = 1, 3, 5, 7
```

Um predicado, nada mais é que uma função (*functor*, *lambda*, ...) que tem como retorno um valor booleano (*true* ou *false*, tipo *bool* no C++). Se estivermos atento, notaremos que para cada algoritmo que desejamos filtrar alguma coisa, deveremos ter uma versão com predicado, correto? Logo duplicamos um algoritmo que atua sobre este par de *iterators* para introduzirmos uma versão com predicado. Correto, C++ optou por isto por questões de eficiência, mas e se tivéssemos *iterators* com predicado? Ou seja, já que iterator é um tipo de abstração de ponteiros, porque não introduzirmos uma certa funcionalidade no *iterator* permitindo que ele “ignore” ou “aceite” certos elementos, tudo isso de uma maneira opaca para o algoritmo? Introduzimos aqui o **iterator_with_predicate** ou o “iterador com predicado”, cujo o beneficio é utilizar algoritmos que suportem *iterator* e não possuem uma versão com predicado:
```cpp
std::vector<int> xs = { 1, 2, 3, 4, 5, 6, 7, 8 }, ys;
ys.resize(xs.size() / 2);
	
auto is_odd = [](int i) {
    return (i & 0x1) == 0x1;
};

auto iwp = make_iterator_with_predicate(xs, is_odd);

std::copy(iwp.begin(), iwp.end(), ys.begin());  //ys = 1, 3, 5, 7
```

Por exemplo, na **STL** não existe um *std::transform_if*, mas é possível adaptar este comportamento com **iterator_with_predicate**:
```cpp
std::vector<int> xs = { 1, 2, 3, 4, 5, 6, 7, 8 };
	
auto is_even = [](int i) {
    return (i & 0x1) == 0x0;
};

auto iwp = make_iterator_with_predicate(xs, is_even);

std::transform(begin(iwp), end(iwp), begin(iwp), [](int i) { return i + 10; }); //xs = 1, 12, 3, 14, 5, 16, 7, 18
```

A idéia principal do **iterator_with_predicate** é oferecer uma visão do *container* que ele atua, uma visão limitada apenas a filtragem de dados, pois a idéia de *view* pode ser mais ampla do que a apresentada aqui.

Em termos de eficiência, conforme no assembly x64 gerado a partir do Visual C++ 14.0 (2015) em Release, a introdução desta abstração (**iterator_with_predicate**) não parece causar grande degradação de desempenho se comparada com uma versão **_if** de um algoritmo da **STL** (46 instruções geradas *inline* contra 30 necessárias para o *std::copy_if*), portanto pode ser uma conveniência interessante para algoritmos que possuem *iterators* e não possuem predicados.
```cpp
std::copy(iwp.begin(), iwp.end(), ys.begin());
00007FF6AE901094  movups      xmm0,xmmword ptr [rbp-29h]  
00007FF6AE901098  movups      xmmword ptr [rbp-29h],xmm0  
00007FF6AE90109C  mov         rax,qword ptr [rbp-19h]  
00007FF6AE9010A0  mov         qword ptr [rbp-29h],rax  
00007FF6AE9010A4  movzx       ecx,byte ptr [rax]  
00007FF6AE9010A7  not         cl  
00007FF6AE9010A9  test        cl,1  
00007FF6AE9010AC  jne         test_efficiency+0D9h (07FF6AE9010D9h)  
00007FF6AE9010AE  lea         rcx,[rax+4]  
00007FF6AE9010B2  mov         rax,rcx  
00007FF6AE9010B5  mov         qword ptr [rbp-29h],rcx  
00007FF6AE9010B9  mov         rdx,qword ptr [rbp-21h]  
00007FF6AE9010BD  cmp         rcx,rdx  
00007FF6AE9010C0  je          test_efficiency+0D9h (07FF6AE9010D9h)  
00007FF6AE9010C2  movzx       ecx,byte ptr [rax]  
00007FF6AE9010C5  not         cl  
00007FF6AE9010C7  test        cl,1  
00007FF6AE9010CA  jne         test_efficiency+0D5h (07FF6AE9010D5h)  
00007FF6AE9010CC  add         rax,4  
00007FF6AE9010D0  cmp         rax,rdx  
00007FF6AE9010D3  jne         test_efficiency+0C2h (07FF6AE9010C2h)  
00007FF6AE9010D5  mov         qword ptr [rbp-29h],rax  
00007FF6AE9010D9  movups      xmm0,xmmword ptr [rbp-29h]  
00007FF6AE9010DD  movaps      xmmword ptr [rbp+17h],xmm0  
00007FF6AE9010E1  mov         r8,qword ptr [rbp+27h]  
00007FF6AE9010E5  cmp         rax,rbx  
00007FF6AE9010E8  je          test_efficiency+129h (07FF6AE901129h)  
00007FF6AE9010EA  mov         rdx,qword ptr [rbp+1Fh]  
00007FF6AE9010EE  mov         rcx,qword ptr [rbp+17h]  
00007FF6AE9010F2  nop         dword ptr [rax]  
00007FF6AE9010F6  nop         word ptr [rax+rax]  
00007FF6AE901100  mov         eax,dword ptr [rcx]  
00007FF6AE901102  mov         dword ptr [r8],eax  
00007FF6AE901105  lea         r8,[r8+4]  
00007FF6AE901109  add         rcx,4  
00007FF6AE90110D  cmp         rcx,rdx  
00007FF6AE901110  je          test_efficiency+124h (07FF6AE901124h)  
00007FF6AE901112  movzx       eax,byte ptr [rcx]  
00007FF6AE901115  not         al  
00007FF6AE901117  test        al,1  
00007FF6AE901119  jne         test_efficiency+124h (07FF6AE901124h)  
00007FF6AE90111B  add         rcx,4  
00007FF6AE90111F  cmp         rcx,rdx  
00007FF6AE901122  jne         test_efficiency+112h (07FF6AE901112h)  
00007FF6AE901124  cmp         rcx,rbx  
00007FF6AE901127  jne         test_efficiency+100h (07FF6AE901100h)

std::copy_if(xs.begin(), xs.end(), zs.begin(), is_even);
00007FF6AE901147  mov         rbx,qword ptr [rbp-39h]  
00007FF6AE90114B  mov         rcx,rbx  
00007FF6AE90114E  mov         r8,qword ptr [rbp-19h]  
00007FF6AE901152  mov         r9,rsi  
00007FF6AE901155  mov         r10,qword ptr [rbp-11h]  
00007FF6AE901159  sub         r10,r8  
00007FF6AE90115C  add         r10,3  
00007FF6AE901160  shr         r10,2  
00007FF6AE901164  cmp         r8,qword ptr [rbp-11h]  
00007FF6AE901168  cmova       r10,rsi  
00007FF6AE90116C  test        r10,r10  
00007FF6AE90116F  je          test_efficiency+193h (07FF6AE901193h)  
00007FF6AE901171  movzx       eax,byte ptr [r8]  
00007FF6AE901175  not         al  
00007FF6AE901177  test        al,1  
00007FF6AE901179  je          test_efficiency+187h (07FF6AE901187h)  
00007FF6AE90117B  mov         rax,rcx  
00007FF6AE90117E  add         rcx,4  
00007FF6AE901182  mov         edx,dword ptr [r8]  
00007FF6AE901185  mov         dword ptr [rax],edx  
00007FF6AE901187  add         r8,4  
00007FF6AE90118B  inc         r9  
00007FF6AE90118E  cmp         r9,r10  
00007FF6AE901191  jne         test_efficiency+171h (07FF6AE901171h)  
00007FF6AE901193  mov         rdi,qword ptr [rbp-31h]  
00007FF6AE901197  sub         rdi,rbx  
00007FF6AE90119A  add         rdi,3  
00007FF6AE90119E  shr         rdi,2  
00007FF6AE9011A2  cmp         rbx,qword ptr [rbp-31h]  
00007FF6AE9011A6  cmova       rdi,rsi
```
Gerados a partir deste exemplo:
```cpp
#include "forward_iterator_with_predicate.hpp"

#include <vector>
#include <algorithm>
#include <iostream>

template <class Iter> 
void display(Iter& iter)
{
	for (auto i : iter) std::cout << i << " ";
	std::cout << "\n";
}

void test_efficiency()
{
	auto is_even = [](int i) {
		return (i & 0x1) == 0x0;
	};

	std::vector<int> xs = { 1, 2, 3, 4, 5, 6, 7, 8 };

	auto iwp = make_iterator_with_predicate(xs, is_even);

	std::vector<int> ys;
	ys.resize(4);

	std::copy(iwp.begin(), iwp.end(), ys.begin());
	
	display(ys);

	std::vector<int> zs;
	zs.resize(4);
	
	std::copy_if(xs.begin(), xs.end(), zs.begin(), is_even);

	display(zs);
}
```
Abaixo, a implementanção em C++ moderno do **iterator_with_predicate** em termos de um *ForwardIterator*:
```cpp
#include <iterator>
#include <functional>

//ForwardIterator Ref.: http://www.cplusplus.com/reference/iterator/ForwardIterator/

template <class Container, class UnaryPredicate>
struct forward_iterator_with_predicate;

template <class Container, class UnaryPredicate>
struct iterator_with_predicate final
{
	iterator_with_predicate(Container& cont, UnaryPredicate pred)
		: fiwp(cont, pred)
	{
		refresh();
	}
	
	//begin and end:
	forward_iterator_with_predicate<Container, UnaryPredicate> begin() const
	{
		forward_iterator_with_predicate<Container, UnaryPredicate> temp = fiwp;		
		temp.update_begin();
		return temp;
	}

	forward_iterator_with_predicate<Container, UnaryPredicate> end() const
	{
		return fiwp;
	}

	void refresh() { fiwp.update_end(); }

private:
	forward_iterator_with_predicate<Container, UnaryPredicate> fiwp;
};

template <class Container, class UnaryPredicate = std::function<bool(typename Container::value_type)>>
auto make_iterator_with_predicate(Container& cont, UnaryPredicate pred) -> iterator_with_predicate<Container, decltype(pred)>
{
	return iterator_with_predicate<Container, decltype(pred)>(cont, pred);
}

template <class Container, class UnaryPredicate = std::function<bool(typename Container::value_type)>>
auto begin(iterator_with_predicate<Container, UnaryPredicate>& iwp) -> decltype(iwp.begin())
{
	return iwp.begin();
}

template <class Container, class UnaryPredicate>
auto end(iterator_with_predicate<Container, UnaryPredicate> iwp) -> decltype(iwp.end())
{
	return iwp.end();
}

template <class Container, class UnaryPredicate>
struct forward_iterator_with_predicate final :
	public std::iterator<std::forward_iterator_tag, typename Container::value_type>
{
	using value_type = typename Container::value_type;

private:
	//constructor and destructor:
	explicit forward_iterator_with_predicate(Container& cont, UnaryPredicate pred) :
		cont(cont),
		pred(pred)
	{
	}

	friend struct iterator_with_predicate<Container, UnaryPredicate>;

public:
	//default-constructible
	forward_iterator_with_predicate() = delete;

	//copy-constructible
	forward_iterator_with_predicate(const forward_iterator_with_predicate& that) = default;

	//copy-assignable
	forward_iterator_with_predicate& operator=(const forward_iterator_with_predicate& that) = default;

	~forward_iterator_with_predicate() = default;

	//comparison:
	//comparable for inequality
	bool operator!=(const forward_iterator_with_predicate& that) const
	{
		return !(*this == that);
	}

	//comparable for equality
	bool operator==(const forward_iterator_with_predicate& that) const
	{
		return cont_first == that.cont_end;
	}

	//dereference:
	//dereferenceable
	value_type& operator*() const
	{
		return *cont_first;
	}

	//dereferenceable
	value_type* operator->() const
	{
		return cont_first;
	}

	//increment:
	//pre increment
	forward_iterator_with_predicate& operator++()
	{
		next();
		return *this;
	}

private:
	void update_end()
	{
		cont_end = cont.end();
	}

	void update_begin()
	{
		cont_first = cont.begin();		
		if (!pred(*cont_first)) //set begin position
			next();
	}

	void next()
	{
		while (++cont_first != cont_end)
		{
			if (pred(*cont_first))
				break;
		}
	}

	Container& cont;
	UnaryPredicate pred;
	typename Container::iterator cont_first, cont_end;
};
```

Fonte:
https://github.com/SimplyCpp/examples/blob/master/forward_iterator_with_predicate_program.cpp
https://github.com/SimplyCpp/examples/blob/master/forward_iterator_with_predicate.hpp