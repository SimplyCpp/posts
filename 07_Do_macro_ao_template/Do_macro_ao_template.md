# Do macro ao template

Uma coisa que gera discussões acirradas dentre as pessoas nestes tempos mais modernos do C++ é o uso de macros.

Dentre os argumentos contra as macros estão os de que o uso de macros leva a criação de uma nova linguagem (o que é bem verdade quando usado __sem moderação__).

Quando se está desenvolvendo em C muitas coisas acabam sendo feitas usando macro por não termos estruturas genéricas para resolver problemas.

Para que a estrutura possa receber int ou double ou seja lá qual tipo numérico, eu __preciso__ usar macro.

```cpp
//Generico. Sem tipos declarados
#define FILL(v, i, start, end, step) { \
	for(i=start; i < end; i+=step) \
		v.push_back(i); \
}

#define ACCUM(v, ret) { \
	for(size_t i=0; i < v.size(); i++) \
		ret += v[i]; \
}
```

A alternativa para C++ é bem conhecida => __templates!__

Os templates fornecem uma forma type-safe, em compile-time, de se implementar as mesmas rotinas, de forma tão eficiente quanto a macro e com validação sintática feita pelo compilador.

No primeiro exemplo, T é o container (vector, list, forward_list, etc) e U é o tipo de dados value_type do container.

Já no accumulate, temos uma __pegadinha:__

```cpp
template<typename T, typename U>
void fill(T &t, U start, U end, U step) {
	for(U i=start; i < end; i += step) {
		t.push_back(i);
	}
}

template<typename T>
auto accum(T &t) { 
	typename T::value_type ret{};
	for(auto &item : t) {
		ret += item;
	}
	return ret;
}
```

Reparem o auto no retorno e a declaração da variável ret. A declaração typename T::value_type é um define interno do container que exporta o tipo de dados que ele aloca internamente. Por não saber se o dado interno é um int ou algo como MeuUltraMegaHugeDouble, __não podemos colocar ret = 0__, mas usamos o inicializador padrão {}.

Bom, claro que temos containers da stdlib para facilitar a nossa vida

```cpp
template<typename T, typename U>
void fill2(T &v, U start, U step) {
	*begin(v) = start;
	generate(begin(v)+1, end(v), [&start, &step]() { return start += step; }); 

	//alternative - this is not a proper std::fill, but a generate
	//for(auto &item : v) {
	//	item = start;
	//	start += step;
	//}
}

template<typename T>
typename T::value_type accum2(T &v) {
	typename T::value_type ret{};

	//auto inside lambda, only in c++14
	for_each(begin(v), end(v), [&ret](auto &val) { ret += val; });
	return ret;
}
```

E só para não dizer que não temos um __bom caso__ para o macro:

```cpp
void log_print(const std::string &str) { /* ... */ }
bool isInfoEnabled() { /* ... */ }

#define LOG_INFO(fmt, x) { \
  if(isInfoEnabled()) log_print(boost::str( boost::format(fmt) % x) ); \
}
```

Desta forma, uma chamada de LOG_INFO(“msg received: %s”, msg->toString()) não resolverá o toString caso o nível de log info não esteja ativo.

Minha regra de ouro pessoal: Se possível evitar a macro, evite.

Código: https://github.com/SimplyCpp/posts/blob/master/07_Do_macro_ao_template/macro_templ.cpp
