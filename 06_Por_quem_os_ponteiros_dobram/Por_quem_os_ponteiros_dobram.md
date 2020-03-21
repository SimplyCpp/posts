# Por quem os ponteiros dobram, estrelando std::accumulate

O **std::accumulate** é um algoritmo de operação numérica, da mesma forma que **std::iota** explorado anteriormente, reside no header **<numeric>** da STL: http://www.cplusplus.com/reference/numeric/accumulate/.

Seu objetivo, até mesmo porque o nome desta função dá uma dica, é acumular elementos que pertencem a um *range* fornecido por um par de *iterators* (usualmente *begin* e *end* para uma sequência completa). O *std::accumulate* também possui um valor inicial para o acumulador que é fornecido como terceiro parâmetro desta função.

Supondo um *array* de tamanho 10 e já inicializado como *container* de referência para os exemplos a seguir:
```cpp
std::array<int, 10> xs { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
```

A utilização padrão do *std::accumulate* com um *container* qualquer (assim como o indicado acima) é da seguinte forma:
```cpp
int initial = 0;
int total = std::accumulate(std::begin(xs), std::end(xs), initial);
std::cout << total << "\n"; //55
```

Onde temos como retorno a soma de todos os elementos do *container*. Com um *container* como o *array* em mãos, logo é possível saber seu tamanho através do membro do tipo função *size*. Assim podemos, por exemplo, extrair a média de um jeito muito simples:
```cpp
std::cout << (std::accumulate(std::begin(xs), std::end(xs), 0.0) / xs.size()) << "\n"; //5.5
```

Perceba que o valor inicial na chamada anterior retornará um tipo *double*, em função do valor inicial fornecido ser deste tipo. Então é fácil concluir, tanto pela assinatura de *std::accumulate* quanto pelo retorno do exemplo anterior, que o tipo inicial e o tipo de retorno serão os mesmos e que o valor do *iterator* quando for derefenciado deverá ser compatível ou convertível para este tipo.

Nos posts passados, mostramos como é possível ter **iterator com predicado** e **alterar o comportamento do operator++ do tipo fornecido para o std::iota**. Podemos juntar estas duas técnicas e fazer o mesmo para o tipo inicial fornecido como terceiro parâmetro da função, como no caso a seguir demonstrado em *accumulator_with_predicate*. Aqui vai o consumo (note que está sendo usado um intervalo/*range* parcial do *container*, e este será filtrado com orientação do predicado indicado):
```cpp
initial = 0;
total = std::accumulate(std::begin(xs), std::begin(xs) + 6, 
                        accumulator_with_predicate<int>(initial, is_even));	
std::cout << total << "\n"; //12
```

E o tipo que oferece um novo significado para o *operator+* e é convertível para o tipo *T* compatível com a variável que recebe o retorno da função:
```cpp
template<typename T>
struct accumulator_with_predicate final
{
	using UnaryPredicate = std::function<bool(T)>;
	
	explicit accumulator_with_predicate(T init, UnaryPredicate pred)
		: acc(init), pred(pred) {}

	accumulator_with_predicate& operator+(const T& val)
	{
		if (pred(val)) acc += val;
		return *this;
	}

	operator T() const { return acc; }

	accumulator_with_predicate() = delete; //default-constructible	
	accumulator_with_predicate(const accumulator_with_predicate&) = default; //copy-constructible
	accumulator_with_predicate& operator=(const accumulator_with_predicate&) = default; //copy-assignable
	~accumulator_with_predicate() = default; //destructor

private:
	T acc;	
	UnaryPredicate pred;
};
```

Caso ache o algoritmo numérico *std::accumulate* verboso para apenas somar elementos (de uma parte) do *container*, sugiro a criação de uma função especialista, como o par de funções *sum* implementadas em termos de *std::accumulate*:
```cpp
template<typename InputIterator>
inline typename InputIterator::value_type sum(InputIterator first, InputIterator last)
{
        typename InputIterator::value_type init{};
	return std::accumulate(first, last, init);
}

template<typename Container>
inline typename Container::value_type sum(const Container& cont)
{
	typename Container::value_type init{};
	return std::accumulate(std::begin(cont), std::end(cont), init);
}
```

Onde o consumo se torna simplificado e extremamente declarativo:
```cpp
total = sum(std::begin(xs), std::begin(xs) + 4);
std::cout << total << "\n"; //10

total = sum(xs);
std::cout << total << "\n"; //55
```

Outro conceito relacionado com *std::accumulate* é o **_fold_** do estilo de programação funcional com *higher-order functions*. O *std::accumulate* (assim como outras funções da biblioteca padrão do C++) é uma *higher-order function*, que tem um *functor* ou objeto do tipo função (ou um *lambda*) como quarto parâmetro a ser informado. Neste caso, é possível mudar o comportamento padrão que é a soma (fornecido pela função *std::plus*) por outra função com operação binária, como por exemplo uma multiplicação. Suponha que seu objetivo é criar uma função *fold* no estilo STL:
```cpp
template<typename InputIterator, typename T, typename Function>
T fold(InputIterator first, InputIterator last, T init, Function f) 
{
	T acc{ init };
	while (first != last)
	{
		acc = f(acc, *first);
		++first;
	}
	return acc;
}
```

Acima temos a implementação de um **_fold left_**, onde a expansão da aplicação consecutiva da função **f** ocorre pela esquerda, algo interpretado na forma: **…f(f(f(f(f(acc, 1),2),3),4),5)…**, se usarmos um *reverse iterator* teremos um **fold right** que pode ser interpretado na forma: **…f(f(f(f(f(acc, 5),4),3),2),1)…**, as duas figuras a seguir, retiradas da página do [Fold](https://wiki.haskell.org/Fold) na [Haskell.org](https://www.haskell.org/), dá uma noção visual do *fold left* (*foldl*) e *fold right* (*foldr*):

![foldl](https://wiki.haskell.org/wikiupload/5/5a/Left-fold-transformation.png)

![foldr](https://wiki.haskell.org/wikiupload/3/3e/Right-fold-transformation.png)

No entanto, não precisamos desta função *fold*, pois o próprio *std::accumulate* é o *fold*! E tudo que é feito com *fold* é possível reproduzir com *std::accumulate* e vice-versa.
```cpp
initial = 0;
total = fold(std::begin(xs), std::begin(xs) + 5, initial, [](int acc, int x) { return acc + x; });
std::cout << total << "\n"; //15

initial = 1;
total = fold(std::begin(xs), std::begin(xs) + 4, initial, [](int acc, int x) { return acc * x; });
std::cout << total << "\n"; //24

initial = 0;
total = std::accumulate(std::begin(xs), std::end(xs), initial, std::plus<int>());
std::cout << total << "\n"; //55
	
initial = 1;
total = std::accumulate(std::begin(xs), std::begin(xs) + 4, initial, [](int acc, int x) { return acc * x; });
std::cout << total << "\n"; //24
```

Dentre as coisas interessantes com *std::accumulate* (ou *fold*), conseguimos calcular um **hash** simples para uma string da seguinte maneira:
```cpp
std::string plain_data = "Simply C++";
size_t seed = 0;
size_t mask = 0x7FFFFFFFFFFFFFFF;
size_t N = 100; //some map to vector/array of size N
size_t plain_data_hash = std::accumulate(std::begin(plain_data), std::end(plain_data), seed,
        	                               [](size_t hash, char c) { return (19 * hash + c) - hash; });
std::cout << "'" << plain_data << "' hash is " << plain_data_hash 
          << " send to slot #" << ((plain_data_hash & mask) % N) << "\n"; 
//'Simply C++' hash is 17691675298189 send to slot #89
```

Vale lembrar que executar um *fold left* ou um *fold right* (*std::accumulate* com o par de iteradores *begin/end* e *std::accumulate* com o par de iteradores *rbegin/rend*, respectivamente) não garantirá uma equabilidade, pois isto dependerá da operação binária e seu tipo possuir a propriedade comutativa. Por padrão a adição de tipos integrais (*short, int, long*) e tipos reais (*float e double*) possuem a propriedade comutativa da adição e da multiplicação:
```cpp
bool equals = std::accumulate(xs.begin(), xs.end(), 0) == std::accumulate(xs.rbegin(), xs.rend(), 0);
std::cout << equals << "\n"; //1 == true
```

Porém, a concatenação de *strings* não possui esta propriedade como podemos ver a seguir:
```cpp
std::array<char, 5> ys{ 'A', 'B', 'C', 'D', 'E' };
	
std::string s_initial = "";
//fold/left fold
std::string s_total = std::accumulate(ys.begin(), ys.end(), s_initial);
std::cout << s_total << "\n"; //ABCDE

s_initial = "";
//fold back/right fold
s_total = std::accumulate(ys.rbegin(), ys.rend(), s_initial);
std::cout << s_total << "\n"; //EDCBA
```

A operação *fold* possui como sua dualidade a operação *unfold*. Note que *fold* (ou *std::accumulate*) retorna um valor combinado, uma espécie de monólito. Imagine que você queira a partir deste bloco obter toda a sequência que o originou. Este é um trabalho para *unfold*, como segue no exemplo com a acumulação da progressão aritmética gerada:
```cpp
std::array<int, 10> zs;
fill_ap(zs.begin(), zs.end(), 2, 3);
display(zs); //2 5 8 11 14 17 20 23 26 29
	
initial = 0;
total = std::accumulate(zs.rbegin(), zs.rend(), initial); //accumulate is fold
std::cout << total << "\n"; //155

std::vector<int> ws;
unfold_ap(total, 2, 3, std::back_inserter(ws));
display(ws); //2 5 8 11 14 17 20 23 26 29
```

```cpp
template<typename InputIterator, typename T>
void fill_ap(InputIterator first, InputIterator last, T first_term, T common_difference)
{
	T n{};
	while (first != last)
	{
		*first = first_term + n * common_difference;
		++n;
		++first;
	}
}

template<typename OutputIterator, typename T>
void unfold_ap(T value, T first_term, T common_difference, OutputIterator result)
{
	T n{};
	while (value > T{})
	{
		T x = first_term + n * common_difference;
		*result = x;
		value -= x;
		++n;
	}
}
```

Como demonstrado neste post, o **std::accumulate**, apesar de ter um propósito bem específico – acumular ou fazer agregação, é uma função bem versátil. Ela também é conhecida popularmente com outros nomes além de **fold**, são eles: **reduce** ou **aggregate**.

Adendo sobre comutatividade e tipos com ponto flutuante (*float* ou *double*):
```cpp
#include <cmath>
#include <limits>

void commutative_and_epsilon()
{
	double a = 1.0, b = 2.0, c = 3.0;
	if (((a + b) + c) == (a + (b + c))) //or: if ((a + b + c) == (c + b + a))
		std::cout << "commutative holds\n";
	else
		std::cout << "commutative NOT holds\n";

	a = 0.1, b = 0.2, c = 3.0;
	if (((a + b) + c) == (a + (b + c))) //or: if ((a + b + c) == (c + b + a))
		std::cout << "commutative holds\n";
	else
		std::cout << "commutative NOT holds\n";

	double x = (a + b) + c, y = a + (b + c); //or: double x = a + b + c, y = c + b + a;
	double delta = std::abs(x - y);
	//double eps = std::numeric_limits<double>::epsilon();
	double eps = (1022 - 52 + 1) * std::pow(2, 52); //http://stackoverflow.com/questions/13698927/compare-double-to-zero-using-epsilon
	if (delta <= eps)
		std::cout << "commutative holds\n";
	else
		std::cout << "commutative NOT holds\n";

	//For more elaborated explanation: 
	//https://randomascii.wordpress.com/2012/02/25/comparing-floating-point-numbers-2012-edition/
}
/*
commutative holds
commutative NOT holds
commutative holds
*/
```

Fonte: https://github.com/SimplyCpp/examples/blob/master/understanding_accumulate_program.cpp
