# Range against the machine

Uma característica do C++ moderno é o **range-based for**. Antes de falar de qualquer teoria e para que fique mais claro, segue um exemplo:

```cpp
for( auto &student : student_list ) {
	student.check_grade();
}
```

O range-based for se baseia em intervalos e iteradores.

O que é um iterador em C++ ? Grosseiramente é um ponteiro que suporte aritmética de ponteiros (Ex: p++, p+=2, p2-p1, etc).

Um intervalo é um conjunto que contém duas extremidades como  elementos, o inicial e o final. Em C++, um intervalo é semi-aberto a  direita, contendo os *iterators* [begin, end)

Para lembrar – o padrão do C++ é sempre com o end aberto.

Agora fica uma pergunta: Se os *containers* do C++ já tem os iteradores, qual seria a razão de eu fazer o meu próprio?

Imagine a seguinte classe:

```cpp
class CashFlow {
	//retorna um intervalo do ano/mês em questão
	getMonthlyFlow(int year, int month)
	//retorna um intervalo do ano/mês em questão
	getYearFlow(int year)
	//retorna um intervalo de um cliente específico
	getClientFlow(const Client &client)
	//retorna um intervalo de um cliente específico, para um período específico
	getClientFlow(const Client &client, int year, int month)
};
```

e isso seria usado da seguinte forma:

```cpp
for( auto &item : cashFlow.getMonthlyFlow(2015, 03) ) {
	//wherever you want to do w/ this data.
}
```

Se você estiver convencido que isso é muito útil, vamos em frente.

Para implementar o seu próprio begin e end, é necessário criar na sua classe um **iterador** e os **métodos de intervalo**

```cpp
class CashFlow {
	using CashMap = multimap<Cash::Date, Cash>;
	using const_iterator = multimap<Cash::Date, Cash>::const_iterator;
	multimap<Cash::Date, Cash> _flow;
	//...
};

template<typename T>
struct CashRange {
	using const_iterator = typename T::const_iterator;        
	const const_iterator begin() const { return _begin; }
	const const_iterator end() const { return _end; }
	//...
};
```

E agora precisamos criar o nosso método de lookup.

```cpp
auto getMonthlyFlow(int year, int month) -> CashRange<CashMap>
{
	int kb = (year * 10000) + (month * 100) + 00;      
	int ke = (year * 10000) + (month * 100) + 31;
	const auto b = _flow.lower_bound(kb);
	if( b == _flow.end() ) {
		return CashRange<CashMap>(_flow.cend(), _flow.cend());
	}
	const auto e = _flow.upper_bound(ke);
	return CashRange<CashMap>(b, e);
}
```

O retorno será um CashRange, contendo um begin/end implementados.
 Agora, para mostrarmos o resultado usaremos o range-based for desta forma:

```cpp
for(auto &c : flow.getMonthlyFlow(2015, 03)) {
	cout << c.second.date << ": " << c.second.value << endl;
}
```

Simples e prático.

Já ouvi a pergunta:
 – Mas usar o iterador não torna o código mais lento ou consome mais memória ?

Esta resposta ficará para um próximo artigo.

Código: https://github.com/SimplyCpp/posts/blob/master/10_Range_against_the_machine/range_for.cpp
