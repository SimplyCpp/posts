# Por quê usar templates ?

Se você não está usando templates com C++, você está perdendo toda a diversão.

A linguagem C++, apesar de suportar orientação a objetos, diferentemente de Java, não te “obriga” a usá-la. Você poderá ter o melhor dos mundos, em termos dos paradigmas e idiomas. Um pouco de orientação a objetos aqui e programação genérica ali, tudo isto no mesmo código!

Vamos fazer uma implementação do seguinte código:

```cpp
string_builder s;
s.add(1).add(12.2).add("thiago ").add(true);
cout << s.str() << endl;
```

Primeiro uma implementação tipicamente orientada a objetos pode ser:

```cpp
class string_builder {
//...
    string_builder &add(int v) {
        std::string s = std::to_string(v);
        _check_size(s.size());
        _s += s;
        return *this;
    }
    string_builder &add(const std::string &v) {
        _check_size(v.size());
        _s += v;
        return *this;
    }
    string_builder &add(const char *v) {
        _check_size(strlen(v));
        _s += v;
        return *this;
    }
    string_builder &add(double v) {
        std::string s = std::to_string(v);
        _check_size(s.size());
        _s += s;
        return *this;
    }
    string_builder &add(bool v) {
        std::string s = v?"Y":"N";
        _check_size(s.size());
        _s += s;
        return *this;
    }
    
    //There are much more methods to implement
    //string_builder &add(unsigned int v);
    //string_builder &add(float v);
    //string_builder &add(long long v);
    //string_builder &add(unsigned long long v);
    //.... And much more ....
```

Vejam só que em uma única classe, para ser de propósito geral e suportar diversos tipos, foram implementados os métodos:

```
1: string_builder &add(int v)
2: string_builder &add(const std::string &v)
3: string_builder &add(const char *v)
4: string_builder &add(double v)
5: string_builder &add(bool v)
6: string_builder &add(unsigned int v)
7: string_builder &add(float v)
8: string_builder &add(long long v)
9: string_builder &add(unsigned long long v)
//Faltando ainda short/char/etc
```

Para uma simples concatenação, temos estas várias implementações, todas quase iguais.

Agora vamos ao template.

O que ele faz ?

O código com template só existe quando é instanciado, ou seja, ele é completamente genérico.
Vamos fazer essa implementação de forma *“templatizada“*:

```cpp
struct string_builder {
  /...
  template<typename T>
  string_builder &add(T v) {
  internal::add<T>(_s, v, _size);
  return *this;
}

namespace internal {
  //General implementation
  template <typename T>
  struct add {
    add(std::string &buffer, T val, int size_inc) {
    std::string s = std::to_string(val);
    _check_size(buffer, s.size(), size_inc);
    buffer += s;
  }
};
```

Bem mais sucinto, não acha ?
Quando o **add** for chamado, o **T** será substituído pelo tipo concreto. No caso da chamada **s.add(1)**, será gerado:

```cpp
string_builder &add(int v) { -> [T = int]
  internal::add<int>(_s, v, _size);
  return *this;
}
```

No entanto, pode surgir a seguinte pergunta: E se eu quiser fazer um tratamento para algum caso específico ? É ai que o template fica ainda mais interessante.

O template tem uma característica muito interessante chamada especialização. Neste caso, é definido o caso genérico com **T**, e também todos os casos específicos, onde um determinado tipo terá seu tratamento diferenciado. Por exemplo, é possível adicionar um **bool**, e ao invés dele virar **0** ou **1**, ele seja projetado com **Y** ou **N**, para **true** e **false** respectivamente.

A especialização ficaria assim:

```cpp
template <>
struct add<bool> {
  add(std::string &buffer, bool val, int size_inc) {
    string s = val?"Y":"N";
    _check_size(buffer, s.size(), size_inc);
    buffer += s;
  }
};
```

E vendo a forma genérica, podemos pensar: Como eu faço para ele tratar o meu próprio tipo ? Se fosse orientado a objetos, seria necessário alterar a classe ou criar uma classe que herdasse dela e usá-la. Isso seria algo do tipo:

```cpp
class my_string_builder : public string_builder {
//...
  string_builder &add(MyClass &v) {
```

Porém, ao invés de fazer isso, podemos simplesmente usar uma especialização de template.

```cpp
//Now I can add my own class to this string_builder add function
struct LazyMessage {
	std::string type;
	int seq;
	std::vector<std::string> entries;
};

namespace Text {
namespace internal {
	template <>
	struct add<LazyMessage> {
		add(std::string &buffer, LazyMessage &msg, int size_inc) {
			std::string s = msg.type;
			for(const auto &entry : msg.entries)
				s += " [" + entry + "]";
			_check_size(buffer, s.size(), size_inc);
			buffer += s;
		}
	};
} //Namespace internal
} //Namespace Text
```

Desta forma o código, além de elegante e sucinto, ficará bem flexível, rápido e divertido.
Parece até que está sendo escrito um código em uma linguagem como Python onde o tipo é sempre inferido.

Fontes:

* https://github.com/SimplyCpp/posts/blob/master/12_Por_que_usar_templates/templ_oo.cpp
* https://github.com/SimplyCpp/posts/blob/master/12_Por_que_usar_templates/templ.cpp
