# Medindo o tempo de seu código

Uma das facilidades do padrão para o C++ moderno é a presença de construções que permitem a manipulação de data e tempo. Estes utilitários podem ser encontrados na biblioteca **chrono**, como você poderá conferir em algumas referências e documentações que se encontram disponíveis, como por exemplo, as minhas favoritas: [Documentação do Visual C++](https://docs.microsoft.com/en-us/cpp/standard-library/chrono?redirectedfrom=MSDN&view=vs-2019), Cplusplus.com e Cppreference.com.

Neste post, vamos focar somente nos utilitários de tempo. O que você pode fazer com eles? É possível saber, através de um de seus relógios (**system_clock**, **steady_clock**, **high_resolution_clock**), sobre um determinado ponto de referência no tempo, como por exemplo, o “agora” através da função estática **now**.

Existe uma série de possibilidades e aplicações para estas classes de tempo, não acha? Uma delas é utilizar um relógio para medir dois pontos distintos no tempo, estabelecendo uma diferença, normalmente denominada de “delta T”. Conforme podemos ver no exemplo a seguir:
```cpp
const auto start = high_resolution_clock::now();
this_thread::sleep_for(seconds(1));
const auto finish = high_resolution_clock::now();
```
O **high_resolution_clock** é ideal para esta tarefa, onde sua escala é baseada em ticks. No entanto, nós queremos saber de uma referência no tempo que possa ser reconhecida por seres humanos, e não numa escala conveniente à máquina, não é mesmo?

Para isso acontecer será necessário fazer um cast da diferença com a unidade de tempo desejada, ou seja, isto requer uma transformação:
```cpp
duration_cast<milliseconds>(finish - start).count()
```

Com o **duration_cast** é possível fazer uma conversão da duração obtida através deste delta para diversas medidas de tempo conhecidas, incluindo segundos, milissegundos, microssegundos, entre outras.

Se você estiver acostumado com os “Stopwatches” existentes em outras bibliotecas para fazer esta tarefa de medição e se sente confortável com eles, porque não encapsular estas funcionalidades da biblioteca **chrono** no seu **stop_watch** caseiro (assim como fizemos e disponibilizamos a seguir)?
```cpp
#include <chrono>

struct stop_watch final
{
    stop_watch() : Start_(now()) {}
    
    std::chrono::seconds elapsed_s() const
    {
        using std::chrono::seconds;
        return std::chrono::duration_cast<seconds>(elapsed());
    }
    
    std::chrono::milliseconds elapsed_ms() const
    {
        using std::chrono::milliseconds;
        return std::chrono::duration_cast<milliseconds>(elapsed());
    }
    
    std::chrono::microseconds elapsed_us() const
    {
        using std::chrono::microseconds;
        return std::chrono::duration_cast<microseconds>(elapsed());
    }
    
    std::chrono::nanoseconds elapsed_ns() const
    {
        using std::chrono::nanoseconds;
        return std::chrono::duration_cast<nanoseconds>(elapsed());
    }
    
    void restart() { Start_ = now(); }
    
    stop_watch(const stop_watch&) = delete;
    stop_watch& operator=(const stop_watch&) = delete;
    
private:
    static std::chrono::high_resolution_clock::time_point now()
    {
        return std::chrono::high_resolution_clock::now();
    }
    
    std::chrono::duration<double> elapsed() const
    {
        return now() - Start_;
    }
    
    std::chrono::high_resolution_clock::time_point Start_;
};
```

Logo, você pode ter uma API sucinta e coveniente para a medição de funções, trechos ou blocos de código, que pode ser consumida da seguinte maneira:
```cpp
stop_watch sw;
...
auto m = sw.elapsed_ms().count();
cout << "Elapsed time is " << m << " milliseconds.\n";

sw.restart()
...
m = sw.elapsed_us().count();
cout << "Elapsed time is " << m << " microseconds.\n";
```

Boa medição com C++ Moderno! ;-)

Fonte: https://github.com/SimplyCpp/examples/blob/master/stop_watch.cpp