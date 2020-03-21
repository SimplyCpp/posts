# Medindo o tempo de seu código

Uma das facilidades do padrão para o C++ moderno é a presença de construções que permitem a manipulação de data e tempo. Estes utilitários podem ser encontrados na biblioteca chrono, como você poderá conferir em algumas referências e documentações que se encontram disponíveis, como por exemplo, as minhas favoritas: Documentação do Visual C++, Cplusplus.com e Cppreference.com.

Neste post, vamos focar somente nos utilitários de tempo. O que você pode fazer com eles? É possível saber, através de um de seus relógios (system_clock, steady_clock, high_resolution_clock), sobre um determinado ponto de referência no tempo, como por exemplo, o “agora” através da função estática now.

Existe uma série de possibilidades e aplicações para estas classes de tempo, não acha? Uma delas é utilizar um relógio para medir dois pontos distintos no tempo, estabelecendo uma diferença, normalmente denominada de “delta T”. Conforme podemos ver no exemplo a seguir:
<script src="https://gist.github.com/8e925e30cde7f8c2fa40"></script>

O high_resolution_clock é ideal para esta tarefa, onde sua escala é baseada em ticks. No entanto, nós queremos saber de uma referência no tempo que possa ser reconhecida por seres humanos, e não numa escala conveniente à máquina, não é mesmo?

Para isso acontecer será necessário fazer um cast da diferença com a unidade de tempo desejada, ou seja, isto requer uma transformação:
<script src="https://gist.github.com/931f7d9505b4ea82bcd1"></script>

Com o duration_cast é possível fazer uma conversão da duração obtida através deste delta para diversas medidas de tempo conhecidas, incluindo segundos, milissegundos, microssegundos, entre outras.

Se você estiver acostumado com os “Stopwatches” existentes em outras bibliotecas para fazer esta tarefa de medição e se sente confortável com eles, porque não encapsular estas funcionalidades da biblioteca chrono no seu stop_watch caseiro (assim como fizemos e disponibilizamos a seguir)?
<script src="https://gist.github.com/3c76023a5da2a2c92dd3"></script>

Logo, você pode ter uma API sucinta e coveniente para a medição de funções, trechos ou blocos de código, que pode ser consumida da seguinte maneira:
<script src="https://gist.github.com/fabiogaluppo/5ee7f21996fbc364506e"></script>

Boa medição com C++ Moderno! ;-)
Fonte: https://github.com/SimplyCpp/examples/blob/master/stop_watch.cpp