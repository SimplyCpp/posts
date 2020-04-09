# STL não tem split – e por que isso não é importante!

Orientação a objetos é um paradigma de programação muito usado nas  linguagens consideradas modernas, tais como: C++, Java, C#, entre  outras… Em teoria, um código orientado a objetos pode ser facilmente  reusado e entendido. Porém, eu gostaria de analisar um caso neste *post*. Então, vamos fazer um pequeno exercício por aqui.

Aviso: [Orientação a objetos](https://en.wikipedia.org/wiki/Object-oriented_programming) (OO) é um paradigma de programação que envolve classes e objetos,  polimorfismo, herança, e uma série de outros recursos. Ter  ou usar objetos não significa que seja orientado a objetos em sua  plenitude. Uma das premissas da OO é que o estado interno dos objetos  deve ser inacessível externamente – isso se chama encapsulamento. No  entanto, encapsulamento do estado sempre é bom? Fica aí algo para  pensarmos, não é mesmo?

São constantes as reclamações que C++ não tem funções básicas na STL como, por exemplo, *string split*. Então, vamos implementar uma por aqui.

Para simplificar, não vamos nos preocupar muito com questões de  eficiência e tratamento de erros, afinal esta (ainda) não é uma  biblioteca real, é só um exercício. 

```cpp
struct nstring {
    nstring(const std::string &s) : _data(s) { }
    const string &data() const { return _data; }
private:
    string _data;
};
```

Temos a nossa classe que encapsula uma *string*. Agora vamos criar um método *split*:

```cpp
const vector<nstring> split(char c) {
    vector<nstring> ret;
    int old = 0;
    for(int i = 0; i < _data.size(); ++i) {
        if( _data[i] == c ) {
            ret.push_back( _data.substr(old, i-old) );
            old=i+1;
        }
    }
    if( old < _data.size() ) { ret.push_back( _data.substr(old) ); }
    return ret;
}
```

Simples e direta. Pega a **string** contida na nossa classe **nstring** e retorna um vetor com n *strings* resultante da separação.

Vamos fazer agora uma implementação não orientada a objetos, apenas  uma função acessando diretamente o estado que na OO seria encapsulado  certamente:

```cpp
vector<string> split(const string &data, char c) {
    vector<string> ret;
    auto old = cbegin(data);
    for(auto it = old; it != cend(data); ++it) {
        if( *it == c ) {
            ret.emplace_back( old, it ); 
            old=it+1;
        }
    }
    if( old != cend(data) ) { 
        ret.emplace_back( old, cend(data) );
    }
    return ret;
}
```

Uma primeira coisa que pode-se pensar é: dentro da classe fica mais  organizado, inclusive por causa dos recursos da IDE (como por exemplo: o IntelliSense ou o Visual Assist), porém vamos imaginar o seguinte:

Essa implementação ficou legal. Vamos colocar na nossa classe *wide_string*. Logo, teríamos duas alternativas se fosse OO:

1. Duplicar a implementação ([copy-and-paste programming](https://en.wikipedia.org/wiki/Copy_and_paste_programming));
1. Criar uma classe base que faça o *split* e outras coisas mais.

```cpp
template<typename T, typename Base>
struct nstring_common {
    vector<Base> split(typename T::value_type c) {
        vector<Base> ret;
        const T &s_data = data();
        // .. same as before ..
    }   
};
 
template<typename T>
struct nstring_basic : public nstring_common<T, nstring_basic<T>> { 
    nstring_basic(const T &value) { _data = value; }
    const T &data() const override { return _data; } 
private:
    T _data;
};
 
using nstring = nstring_basic<string>;
using nwstring = nstring_basic<wstring>;
```

No caso da versão não OO, não precisamos fazer nada, apenas deixá-la [templatized](http://www.urbandictionary.com/define.php?term=templatized)! Ou seja, genérica:

```cpp
template<typename T>
vector<T> split(T &data, typename T::value_type c) {
    vector<T> ret;
    auto old = cbegin(data);
    for(auto it = old; it != cend(data); ++it) {
        if( *it == c ) {
            ret.emplace_back( old, it ); 
            old=it+1;
        }
    }
    if( old != cend(data) ) { 
        ret.emplace_back( old, cend(data) );
    }
    return ret;
}
```

Só que eu gostei tanto desta versão de *split* que eu poderia usá-la para um outra estrutura com requisitos compatíveis, ao invés da *string,* porque não utilizar com um vetor?

Agora eu tenho outras duas alternativas:

1. Novamente duplicar a implementação ([copy-and-paste programming](https://en.wikipedia.org/wiki/Copy_and_paste_programming));
1. Criar uma classe base **sequential_buffer** implementando o *split*.

Neste caso a implementação fica de lição de casa!

No caso da versão não OO, não precisamos fazer nada:

```cpp
using my_type = vector<int>;  
my_type x = {1, 2, 3, 4, 5, 6, 4, 7, 8, 4, 9, 10};
vector<my_type> xs = split(x, 4);
```

Agora vamos supor que eu estou tão animado com a minha implementação de *split* que eu quero definir um predicado.

Vou fazer só na minha versão genérica, pois já mostrei um dos motivos da STL não ser orientada a objetos: Ser extensível mais facilmente!

```cpp
template<typename T>
vector<T> split(T &data, typename T::value_type c, 
                std::function<bool(typename T::value_type, typename T::value_type)> pred) {
    vector<T> ret;
    auto old = cbegin(data);
    for(auto it = old; it != cend(data); ++it) {
        if( pred(*it, c) ) {
        //...
}
```

Primeiro vamos manter o comportamento anterior, onde a separação é  baseada no número 4. Para isto, utilizamos um predicado de igualdade. No caso, o predicado é uma função que retorna um *bool* – ela permitirá a mudança do comportamento do algoritmo de *split*.

```cpp
my_type x = {1, 2, 3, 4, 5, 6, 4, 7, 8, 4, 9, 10};
vector<my_type> xs = split(x, 4, [](int a, int b) {
    return( a == b );
}); 
print(xs);
```

Agora, com uma pequena alteração no predicado, e tornar o  exemplo mais divertido, vamos separar os elementos pares e deixar os  ímpares!

```cpp
my_type y = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12};
vector<my_type> ys = split(y, 2, [](int a, int b) {
    return( a % b == 0 );
});
```

Finalizando, o intuito aqui não é criar polêmica ou dizer que a  orientação a objetos é ruim e a programação genérica é boa. Mas sim,  mostrar como um paradigma, quando levado a risca, de forma rígida,  poderá mais atrapalhar do que ajudar.

### Links

Fontes:

- [oo1.cpp](https://github.com/SimplyCpp/posts/blob/master/19_STL_nao_tem_split_e_por_que_isso_nao_e_importante/oo1.cpp)
- [noo1.cpp](https://github.com/SimplyCpp/posts/blob/master/19_STL_nao_tem_split_e_por_que_isso_nao_e_importante/noo1.cpp)
- [oo2.cpp](https://github.com/SimplyCpp/posts/blob/master/19_STL_nao_tem_split_e_por_que_isso_nao_e_importante/oo2.cpp)
- [noo2.cpp](https://github.com/SimplyCpp/posts/blob/master/19_STL_nao_tem_split_e_por_que_isso_nao_e_importante/noo2.cpp)
- [noo3.cpp](https://github.com/SimplyCpp/posts/blob/master/19_STL_nao_tem_split_e_por_que_isso_nao_e_importante/noo3.cpp)
- [noo4.cpp](https://github.com/SimplyCpp/posts/blob/master/19_STL_nao_tem_split_e_por_que_isso_nao_e_importante/noo4.cpp)