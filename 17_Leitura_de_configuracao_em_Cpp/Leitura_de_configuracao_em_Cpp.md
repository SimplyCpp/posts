# Leitura de configuração em C++

Uma coisa que é comum no ambiente Java e que eu gosto muito são os arquivos de *properties.* Não é de hoje que eu os uso para configurar aplicações que eu faço.
 Eu tinha uma classe de configuração feita na época do C++98 e que hoje,  usando, fiquei com vontade de reescrevê-la para ficar com um aspecto  mais atual.

Eu pensei nas seguintes regras:

1. Eu não preciso manter a mesma assinatura;
2. Vou tentar fazer da forma mais didática possível;
3. Vou tentar não repetir código;
4. Eu quero me divertir fazendo isso;
5. Vamos falar de ***Policy-based design***.

Antes de começar, vamos falar um pouco sobre o ***Policy-based design***. O que é ?
 Este é um paradigma onde, basicamente, nós trocamos todas as  implementações concretas por tipos genéricos. Isso provê uma grande  flexibilidade às bibliotecas que construímos.

Primeira coisa é definir como seria a estrutura que armazenaria as configurações:

```cpp
template<typename Key = std::string, typename Value = std::string, typename Store = std::unordered_map<Key, Value>>
struct config_holder {
	using key_iter = typename Key::const_iterator;
	using val_iter = typename Value::const_iterator;
	using key_pair = std::pair<key_iter, key_iter>;
	using val_pair = std::pair<val_iter, val_iter>;
	using range = std::pair<key_pair, val_pair>;
	using map_value = typename Store::value_type;
};
```

Aqui vemos as nossas primeiras *policies* – **Key**, **Value** e **Store**. Elas tem valor *default*, mas poderiam ser quaisquer outras classes.

Temos a nossa estrutura. Vamos agora ler o arquivo de configuração.
 A leitura de um arquivo *properties* é bem simples. Vamos ler linha a linha
 e exibir os valores:

```cpp
std::ifstream istr;
istr.open(file_name);

while( istr.good() )  {
	std::string line;
	std::getline(istr, line);
	std::cout << line << std::endl;
}
```

Agora já temos as nossas linhas do arquivo sendo mostradas na tela.
Antes de adicionar no *config_holder*, precisamos fazer um *parsing* simples para separar o par **key=value**. Agora começa um pouco de diversão.

Vamos trocar o *cout* por uma função *crack* que vai quebrar a linha e gerar um par com chave/valor.

```cpp
template<typename Key, typename Value, typename AdderFunc>
void crack(config_holder<Key, Value> &config, const std::string &line, char delim, AdderFunc f) {
	auto b = std::begin(line);
	auto e = std::end(line);
	auto pos = line.find(delim);
	if( pos != std::string::npos ) {
		f(config, b, b+pos, e);
	}
}

//nossa AdderFunc
void default_adder(
	config_holder<std::string, std::string> &config, 
	iter_type b, iter_type m, iter_type e) {
	
	//b-m => key / m+1-e => value
	config.add_item( b, m, m+1, e );
}
```

Nós temos aqui um método que vai procurar por um delimitador e retornar os iteradores de início, meio e final, sendo:

- Início = começo da *string* (b);
- Meio = posição do delimitador (pos);
- Final = final do valor (e).

Ou seja, temos os respectivos intervalos para **chave => [b, b+pos)** e **valor => (b+pos, e)**. E temos uma função *f* do tipo *AdderFunc* que irá receber os iteradores para jogar os valores no mapa.
 Repare que não estamos fazendo cópias da *string*, mas apenas passando os iteradores.

E agora precisamos somente implementar as funções de busca da classe de configuração. Eu pensei nas seguintes funções:

- *get(key)* -> retorna valor de key
- *prefix(prefix)* -> retorna valores com o prefixo prefix
- *after(prefix)* -> retorna o resto das chaves começadas em prefix
- *next_token(prefix, separator)* -> retorna o próximo *token.* Ex: *plugin.name*

Vamos agora ver a implementação delas.

A função *get* simplesmente pega o mapa e retorna o valor da chave indicada

```cpp
auto get(const Key &key) const -> const Value & {
	auto it = _items.find(key);
	if( it == _items.end() )
		return _empty;
	return it->second;
}
```

Antes, para implementarmos as outras funções, precisamos fazer uma função auxiliar. Esta função tem como objetivo:

- iterar pelo mapa e comparar se o prefixo é igual ao passado
- caso seja, invocar um ***callback*** passando o par chave/valor como parâmetro (Assim não teremos cópia)

```cpp
using adder_func = std::function<void(const std::pair<Key,Value> &pair)>;
void iterate_and_check(const Key &prefix, adder_func f_adder) const {
	std::for_each(begin(_items), end(_items), [&prefix, &f_adder](const std::pair<Key,Value> &pair) {
		//let's check prefix
		if( std::equal( begin(prefix), end(prefix), begin(pair.first) ) )
			f_adder(pair);
	});
}
```

Usando a nossa função auxiliar, a função *prefix* fica muito mais simples:

```cpp
auto prefix(const Key &prefix) const -> std::vector<map_value> {
	std::vector<map_value> values;
	iterate_and_check(prefix, [&values](const std::pair<Key,Value> &pair) {
		values.push_back(pair);				
	});
	return values;
}
```

As funções *after* e *next_token* são pequenas variações da função *prefix.*

Na primeira, faremos um *substring* para pegar o resto da chave e na segunda somente o *token* seguinte.

```cpp
auto after(const Key &prefix) const -> std::unordered_set<Key> {
	std::unordered_set<Key> values;
	iterate_and_check(prefix, [&values, &prefix](const std::pair<Key,Value> &pair) {
		values.insert( pair.first.substr( prefix.size() ) );
	});
	return values;
}

auto next_token(const Key &prefix, const typename Key::value_type separator) const -> std::unordered_set<Key> {
	std::unordered_set<Key> values;
	int pos = last_of(prefix, separator);
	iterate_and_check(prefix, [&values, &prefix, &pos, &separator](const std::pair<Key,Value> &pair) {
		int npos = next_of(pair.first, pos, separator);
		values.insert( pair.first.substr( pos, npos ) );
	});
	return values;
}
```

E vamos mostrar agora como usar a nossa classe de configuração:

```cpp
//Leitura de uma chave
const std::string &log_dir = cfg.get("log.base_dir");

//Agora vamos pegar todas as configurações relativas a log
auto plugins = cfg.prefix("log.");
for(auto p : plugins)
	std::cout << p.first << "=>" << p.second << std::endl;

//Lista de plugins na configuração
auto tok = cfg.next_token("plugin.", '.');
for(auto p : tok) {
	std::cout << p << std::endl;
}
```

Como exemplo de uma **mudança de policy**, vamos trocar o **Value** de *string* para *my_value.*

```cpp
struct my_value {
	std::string sval;
	using const_iterator = std::string::const_iterator;
};
//...
auto cfg = configuration::load<std::string, my_value>(fname, 
	[](auto &config, iter_type b, iter_type m, iter_type e) {
		std::string k(b, m);
		my_value mv;
		mv.sval = std::string(m+1, e);
		config.add_item( k, mv );
	});
```

Tivemos basicamente que adicionar uma nova classe e seu *parser*.

Este *post* foi um pouco mais extenso, onde apenas arranhamos a superfície do **Policy-based design**. Espero que tenham gostado.

Sobre ***Policy-based design***:

[Policy-based design - Wikipedia](https://en.wikipedia.org/wiki/Policy-based_design)

Fontes:
- [config.cpp](https://github.com/SimplyCpp/posts/blob/master/17_Leitura_de_configuracao_em_Cpp/config.cpp)
- [config.h](https://github.com/SimplyCpp/posts/blob/master/17_Leitura_de_configuracao_em_Cpp/config.h)
- [example.cfg](https://github.com/SimplyCpp/posts/blob/master/17_Leitura_de_configuracao_em_Cpp/example.cfg)