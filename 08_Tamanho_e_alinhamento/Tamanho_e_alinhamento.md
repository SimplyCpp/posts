Uma coisa que eu gosto bastante no C++ e que poucas vezes é visto com o devido cuidado é a forma como a memória é alocada. Eu ouço constantemente o mantra: Hoje temos muita memória disponível, um pouco a mais não faz diferença.

Opa, perai.

É verdade que é comum encontrarmos máquinas com 16, 32, 48 GB de memória, mas quando se fala de performance é também importante usar o mínimo possível de memória para um melhor aproveitamento do __cache do processador.__

Vejamos um caso de memória perdida por causa de alinhamento do processador.

Temos a struct:

```cpp
struct Field {
	unsigned int f1;  //int=4 bytes
	unsigned char f2; //char=1 byte
};
```

Temos nela um int de 4 bytes + char de 1 byte = 5 bytes. Ah, quase esqueci, temos mais 3 bytes de __alinhamento.__

```
int:0-4, char:4-5 => Total bytes: 8
0 0 0 0 0 ff ff ff  Wasted bytes: 3
```

Isso acaba não sendo perceptível também nos casos onde a perda ocorre no meio da struct

```cpp
struct FieldC {
	unsigned char f1; //char=1 byte
	unsigned int f2;  //int=4 bytes
};
```

```
char:0-1, int:4-8 => Total bytes: 8
0 ff ff ff 0 0 0 0  Wasted bytes: 3
```

Tal qual o exemplo anterior, foram perdidos 3 bytes nesse exemplo, ou seja, 37,5% de espaço perdido. Um descuido no layout de uma struct pode ser até catastrófico!

Vejamos o exemplo:

```cpp
struct Field4 {
	unsigned char f1; //char=1 byte
	unsigned int f2;  //int=4 bytes
	unsigned char f3; //char=1 byte
};
```

```
char:0-1, int:4-8, char:8-9 => Total bytes: 12
0 ff ff ff 0 0 0 0 0 ff ff ff  Wasted bytes: 6
```

Tivemos no exemplo acima 6 bytes perdidos, 50% do tamanho da struct. Isso pode fazer muita diferença quando um cálculo de uso de memória subir de 2 GB para 4 GB (sem contar fragmentação de memória e custo do alocador new).

A solução para isso é bem simples. Uma melhor disposição da struct resolve o problema.

```cpp
struct Field5 {
	unsigned int f2;  //int=4 bytes
	unsigned char f1; //char=1 byte
	unsigned char f3; //char=1 byte
};
```

```
int:4-5, char:0-4, char:5-6 => Total bytes: 8
0 0 0 0 0 0 ff ff  Wasted bytes: 2
```

Veja só ! Metade do tamanho.

Eu poderia ter nesse cálculo também mais 4 ou 8 bytes de um __ponteiro para uma vtable__, mas esse é assunto para outro post.

Código: https://github.com/SimplyCpp/posts/blob/master/08_Tamanho_e_alinhamento/sizes.cpp
