# Construtor e Operador =

É bem comum que as pessoas não entendam e nem saibam a razão de existir o copy/move constructor e assignment operator.
 Por padrão, classes no C++ são do tipo **value_type**. Isso faz com que ao atribuir algo a uma variável de classe, uma cópia  de ponteiros não possa ser feita pela classe estar localizada na stack.
 O operador de cópia e movimentação faz com que não sejam gastos ciclos desnecessários pelo programa.

**Exemplo comentado:**

Constructors e assignment operator

```cpp
//Construtor padrão
TestCopy() {
  cout << "[CONSTR 1]" << endl;
  this->val = 0;
}

//Construtor parametrizado
TestCopy(T val) {
  cout << "[CONSTR 2:" << val << "]" << endl;
  this->val = val;
}

//Copy constructor
TestCopy(const TestCopy &src) {
  cout << "[COPY]" << endl;
  this->val = src.val;
}

//Move constructor – Note que não há const e tem dois “&” na assinatura
TestCopy(TestCopy &&src) {
  cout << "[MOVE]" << endl;
  this->val = std::move(src.val);
}

//Operador = (atribuição)
TestCopy &operator =(const TestCopy &src) {
  cout << "[OP=" << src.val << "]" << endl;
  this->val = src.val;
}

//Final do programa + função auxiliar
void ts() {
  cout << val << "===============" << endl;
}
```

Agora segue a execução.

Repare que o **assignment operator não está sendo chamado** para as variáveis abaixo, por estarem todos sendo construídos agora.

```cpp
                                //Saída do programa:
TestCopy<> t1;                  //[CONSTR 1]
TestCopy<> t2 {1};              //[CONSTR 2:1]
TestCopy<> t3 = t2;             //[COPY]
TestCopy<> t4 {t2};             //[COPY]
TestCopy<> t5 = TestCopy<>{5};  //[CONSTR 2:5]
TestCopy<> t6 = {6};            //[CONSTR 2:6]
TestCopy<> t7 = 7;              //[CONSTR 2:7]
```

Já neste segundo bloco, os objetos que receberam a atribuição são **pré-existentes**. Não há de se chamar um construtor em um objeto **já construído**, certo?
Desta forma, o operador = é invocado para que os valores sejam copiados para dentro do objeto à esquerda.

```cpp
t5.ts();            //5===============
t1 = t3;            //[OP=1]
t1.ts();            //1===============
t1 = TestCopy<>{3}; //[CONSTR 2:3]
                    //[OP=3]
t3.ts();            //1===============
```

No terceiro e último bloco, o objeto TestCopy<>(99) existe **apenas no escopo interno** do push_back. Isso faz com que o Move constructor seja invocado.

```cpp
vector<TestCopy<>> tm;          
tm.push_back(TestCopy<>(99));   //[CONSTR 2:99]
                                //[MOVE]
tm[0].ts();                     //99===============
```

**Regra simples para diferenciar:**

- Se objeto não existe ainda => Constructor
- Se objeto já existe => Operator

Código: https://github.com/SimplyCpp/posts/blob/master/02_Construtor_e_Operador_igual/constructor.cpp