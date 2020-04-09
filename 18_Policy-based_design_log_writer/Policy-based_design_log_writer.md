# Policy-based design: log writer

**Policy-based design**

Vamos neste artigo dar mais uma pincelada no **Policy-based design**. Vamos fazer como exemplo uma classe de *log.*
 Como este é só um exemplo, não vamos considerar múltiplos parâmetros no *log,* mas somente uma *string,* assim não fugiremos do assunto.

Uma das coisas mais importantes neste tipo de *design* é o desacoplamento. Ele é uma excelente alternativa ao uso de interfaces por duas razões:

1. Não gera chamadas virtuais (ou um nível de indireção em tempo de execução)
    *Duck typing* (https://pt.wikipedia.org/wiki/Duck_typing)
2. Eu gosto bastante desse tipo de *design,* já usado aqui: http://simplycpp.com/2016/02/05/leitura-de-configuracao-em-c/

Vamos ver o seguinte exemplo:
(Sem *variadic templates* e parâmetros genéricos para não fugir do tema)

```cpp
struct log_writer {
	void error(const std::string &s) {
		cout << s << endl;
	}
	void info(const std::string &s) {
		cout << s << endl;
	}
};
//...
log_writer logger;
logger.info("Starting...");
```

Este código é simples e funciona bem, mas vemos de cara um acoplamento – cout.

A primeira alteração que devemos fazer é uma **policy** para a escrita dos logs – **writer_policy**

```cpp
//Concept: Writer must define operator <<
#define Writer typename
 
template<Writer writer_policy>
struct log_writer {
    writer_policy& _writer;
    log_writer(writer_policy& writer) : _writer(writer) { }
 
    void error(const std::string& s) {
        _writer << s;
    }
    void info(const std::string& s) {
        _writer << s;
    }
};
```

Agora temos uma *policy* onde o *writer* precisa definir o operador **<<** somente.
Inicialmente, duas *policies* naturais para o nosso caso são uma que imprima o *log* no console e outra que imprima o *log* em arquivo:

```cpp
struct console_writer {
    template<typename T>
    console_writer& operator << (const T& s) {
        std::cout << s << endl;
        return *this;
    }
};
 
struct file_writer {
    ofstream _file_s;
    file_writer(const string& f_name) {
        _file_s.open(f_name); //Example - never fails
    }
    ~file_writer() {
        _file_s.close();
    }
 
    template<typename T>
    file_writer& operator << (const T& s) {
        _file_s << s << endl;
        return *this;
    }
};
```

E para usarmos agora o nosso *log_writer* com uma *policy,* fazemos da seguinte forma:

```cpp
console_writer f;
log_writer<console_writer> logger(f);
// ou
file_writer f(file_name);
log_writer<file_writer> logger(f);
```

Bem simples e fácil.

De uma forma transparente para o algoritmo de *log,* temos a abstração da saída de *log* para arquivo ou console.

Uma alternativa para isso é usar uma interface (com membros do tipo  funções virtuais pura) e uma implementação. Isso nos causa problemas:

1. Para que a classe possa ser usada, ela precisa necessariamente herdar de uma interface específica;
2. Usando o *duck typing*, o importante não é a herança, mas sim a assinatura dos métodos respeitar o uso;
3. Com uma interface e duas implementações, teremos uma chamada em *vtable* sempre que os métodos forem chamados.

Isso será discutido mais profundamente em outro artigo.

Continuando, uma outra *policy* interessante de ser implementada é a formatação de *log.*
 Um exemplo seria fazer uma formatação como:
 **[2016-03-01 09:50:47] Starting…**

Vamos alterar o *log_writer* para suportar formatação.

```cpp
template<Writer writer_policy, Formatter formatter_policy>
struct log_writer {
    writer_policy& _writer;
    formatter_policy _formatter;
 
    log_writer(writer_policy& writer, formatter_policy formatter) :
        _writer(writer), _formatter(formatter) { }
 
    void error(const std::string& s) {
        _writer << _formatter(s);
    }
    void info(const std::string& s) {
        _writer << _formatter(s);
    }
};
```

Eu recebi um **formatter_policy**, sendo que este precisa definir o operador **()** e retornar uma *string.*

Agora vamos implementar o *formatter:*

```cpp
std::string dt_format(const string& s) {
    //...
    timeinfo = localtime(&rawtime);
    strftime(buffer, sizeof(buffer), "[%Y-%m-%d %I:%M:%S] ",timeinfo);
    std::string str(buffer);
    return str + s;
}
struct my_formatter {
    std::string operator()(const std::string& s) {
        return dt_format(s);
    }
};
```

Um detalhe interessante. Como podemos usar o *formatter* ?
Para nós, tanto uma função quanto um *Functor* vai funcionar.

Vamos mostrar um exemplo de cada abaixo.

```cpp
// usando uma função simples
log_writer<console_writer, std::function<string(const string&)>> logger(out_writer, dt_format);
// 2a forma
log_writer<console_writer, std::function<decltype(dt_format)>> logger(out_writer, dt_format);
// Ou usando um functor
my_formatter fmt;
log_writer<console_writer, my_formatter> logger(out_writer, fmt);
```

Uma outra alternativa interessante é usar um *lambda.*

```cpp
auto l_fmt = [](const string& s) -> string {
    return "--> " + s;
};
log_writer<console_writer, decltype(l_fmt)> logger(out_writer, l_fmt);
```

Um último ponto a se ver é a legibilidade. Podemos criar uma função onde os tipos são inferidos e não precisam ser declarados.

Podemos fazer um *helper make_logger.* Vamos ver como fica o uso:

```cpp
template<Writer writer_policy, Formatter formatter_policy>
auto make_logger(writer_policy& writer, formatter_policy formatter) -> log_writer<writer_policy, formatter_policy>
{
    return log_writer<writer_policy, formatter_policy>(writer, formatter);
}
 
//Usage:
auto logger = make_logger(out_writer, dt_format);
```

Bem melhor, não ?

Fontes:

- [policy1.cpp](https://github.com/SimplyCpp/posts/blob/master/18_Policy-based_design_log_writer/policy1.cpp)
- [policy2.cpp](https://github.com/SimplyCpp/posts/blob/master/18_Policy-based_design_log_writer/policy2.cpp)
- [policy3.cpp](https://github.com/SimplyCpp/posts/blob/master/18_Policy-based_design_log_writer/policy3.cpp)
- [policy4.cpp](https://github.com/SimplyCpp/posts/blob/master/18_Policy-based_design_log_writer/policy4.cpp)