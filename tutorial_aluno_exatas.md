
# Tutorial para o módulo *aluno_exatas*

Esse tutorial tem como objetivo explorar os módulos contidos no módulo `aluno_exatas`.

Atualmente, os módulos disponíveis são:
* [`aluno_exatas.fis_exp`](#aluno_exatas.fis_exp)
* [`aluno_exatas.calc_num`](#aluno_exatas.calc_num)
* [`aluno_exatas.circuitos_eletricos`](#aluno_exatas.circuitos_eletricos)

## *aluno_exatas.fis_exp*

Esse módulo tem como objetivo auxiliar o aluno que está cursando física experimental. Além de automatizar a propagação, permite gerar funções que facilitam a manipulação dos dados medidos.

### *aluno_exatas.fis\_exp.FisExp*

Essa classe é útil principalmente na propagação de incertezas.

#### Importando módulos úteis:


```python
import aluno_exatas.fis_exp as fe
import numpy as np
```

#### Inicializando o módulo

A variável `f` irá conter um objeto `FisExp` cuja função principal é `a+b*c`. Nesse objeto, pode ser achada a propagação de incertezas da função principal.


```python
f = fe.FisExp('a**2+b*c')

print (f'Função principal: {f.funcao}')
print (f'Variáveis da função principal: {f.variaveis}\n')
print (f'Propagação de incertezas da função principal: {f.propagacao}')
print (f'Incertezas da propagação de incertezas: {list(f.incertezas.values())}')
```

    Função principal: a**2 + b*c
    Variáveis da função principal: {c, b, a}
    
    Propagação de incertezas da função principal: sqrt(4*a**2*u_a**2 + b**2*u_c**2 + c**2*u_b**2)
    Incertezas da propagação de incertezas: [u_c, u_b, u_a]


#### Definindo valores conhecidos

Nesse caso, os valores de `a` e `b` são constantes, assim como as incertezas de `a` e `c`. Os valores desconhecidos são `c` e a incerteza de `b`.


```python
f.valores_conhecidos = {'a':4, 'b':2}
f.incertezas_conhecidas = {'a':1, 'c':2}

print (f'Função com valores constantes substituídos: {f.funcao_substituida}')
print (f'Propagação com valores constantes substituídos: {f.propagacao_substituida}')
```

    Função com valores constantes substituídos: 2*c + 16
    Propagação com valores constantes substituídos: sqrt(c**2*u_b**2 + 80)


#### Criação de funções

Agora são criadas funções para calcular o valor da função principal e da propagação em diferentes valores de `c` e `u_b`.


```python
f.gerar_funcao(['c'])
f.gerar_propagacao(['c','u_b'])

print (f'Função principal avaliada em c=5: {f.funcao_gerada(5)}')
print (f'Propagação avaliada em c=5, u_b=1: {f.propagacao_gerada(5,1)}')
```

    Função principal avaliada em c=5: 26
    Propagação avaliada em c=5, u_b=1: 10.246950765959598


Essas funções também podem ser utilizadas com valores armazenados em `numpy.arrays`, o que permite que sejam avaliadas em vários pontos. Quando os parâmetros das funções são vários `numpy.arrays`, a função é avaliada de forma sequencial, seguindo a sequência de cada array (portanto, os vetores precisam ter o mesmo tamanho).


```python
c = np.linspace(0,50,5)
u_b = np.array([1,1,2,2,1])

print (f'Função principal avaliada em diferentes valores de c: {f.funcao_gerada(c)}')
print (f'Propagação avaliada em diferentes valores de c e u_b: {f.propagacao_gerada(c, u_b)}')
```

    Função principal avaliada em diferentes valores de c: [ 16.  41.  66.  91. 116.]
    Propagação avaliada em diferentes valores de c e u_b: [ 8.94427191 15.37042615 50.7937004  75.5314504  50.7937004 ]


#### Funcionalidades extra

Se, por alguma razão, o usuário desejar integrar ou derivar a função principal, há funções que permitem isso.


```python
print ('Função derivada em relação a "a": ', f.derivar('a'))
print ('Função derivada em relação a "a", avaliada em a=1, derivada de índice 2: ', f.derivar('a', ponto_avaliado=1, indice=2))
print ('Função derivada em relação a "a", com valores conhecidos substituídos: ', f.derivar('a', substituir=True), '\n')

print ('Integral da função em relação a "a": ', f.integrar('a'))
print ('Integral da função em relação a "a", avaliada entre [0,5]: ', f.integrar('a', limites=[0,5]))
print ('Integral da função em relação a "c", avaliada entre [0,5], com valores conhecidos substituídos: ', f.integrar('c', limites=[0,5], substituir=True))
```

    Função derivada em relação a "a":  2*a
    Função derivada em relação a "a", avaliada em a=1, derivada de índice 2:  2
    Função derivada em relação a "a", com valores conhecidos substituídos:  8 
    
    Integral da função em relação a "a":  a**3/3 + a*b*c
    Integral da função em relação a "a", avaliada entre [0,5]:  5*b*c + 125/3
    Integral da função em relação a "c", avaliada entre [0,5], com valores conhecidos substituídos:  105


### *aluno_exatas.fis\_exp.MMQ*

Essa classe foi feita para facilitar a utilização do MMQ linear, com incertezas variáveis ou não.

#### Importação de módulos

O módulo `fis_exp` já foi importado, de forma que não é necessário importá-lo novamente.

#### Inicializando o módulo

O módulo pode ser inicializado de diferentes formas, dependendo de que tipo de MMQ se quer calcular.


```python
mmq_inc_iguais = fe.MMQ()
mmq_inc_varia = fe.MMQ(fe.Incertezas.variaveis)
```

#### Definição de valores

Os módulos podem receber `lists`, `numpy.ndarrays` ou números normais (no caso de `incerteza_y` para incertezas iguais).


```python
x = [0, 1, 2, 3, 4]
y = np.array([1, 3, 7, 8, 10])

mmq_inc_iguais.x_values = x
mmq_inc_iguais.y_values = y
mmq_inc_iguais.incertezas_y = 1

mmq_inc_varia.x_values = x
mmq_inc_varia.y_values = y
mmq_inc_varia.incertezas_y = [1, 1, 2, 1, 3]
```

#### Recebendo de volta os coeficientes

Os coeficientes da função na forma `y = a + b*x` são retornados no formato `[a,b]`, assim como as incertezas.


```python
coef_inc_iguais, incert_inc_iguais = mmq_inc_iguais.coeficientes()
coef_inc_varia, incert_inc_varia = mmq_inc_varia.coeficientes()

print ('Coeficientes da função com incerteza constante: ', coef_inc_iguais)
print ('Incertezas dos coeficientes da função com incerteza constante:', incert_inc_iguais, '\n')

print ('Coeficientes da função com incerteza variável: ', coef_inc_varia)
print ('Incertezas dos coeficientes da função com incerteza variável:', incert_inc_varia)
```

    Coeficientes da função com incerteza constante:  [1.2 2.3]
    Incertezas dos coeficientes da função com incerteza constante: [0.77459667 0.31622777] 
    
    Coeficientes da função com incerteza variável:  [1.01265823 2.36075949]
    Incertezas dos coeficientes da função com incerteza variável: [0.65822785 0.14556962]


## *aluno_exatas.calc_num*

Esse módulo tem como objetivo auxiliar o aluno que está cursando cálculo numérico. Contém funções que implementam conceitos aprendidos em aula.

#### Importando módulos úteis:


```python
import aluno_exatas.calc_num as cn
import numpy as np
```

### Métodos de aproximações sucessivas para achar zero de funções

#### Definindo a função que será usada e sua derivada

A definição da derivada é necessária para o método de Newton.


```python
def f(x):
    return x + np.cos(x)

def flinha(x):
    return 1 - np.sin(x)
```

#### Definindo aproximações iniciais para os métodos da secante, newton e bissecção

* O método da secante precisa apenas da própria função e de duas aproximações para a raiz;
* O método de Newton precisa da função, sua derivada, e uma aproximação inicial;
* O método da bissecção precisa própria função e de duas aproximações para o intervalo que contém a raiz.


```python
x_secante = 0.0
xo_secante = 1.0

x_newton = 1.0

a_bi = -1.0
b_bi = 0.0

print('Função inicial avaliada nas aproximações iniciais:')
print(f'f(-1) = {f(-1)}')
print(f'f(0) = {f(0)}')
print(f'f(1) = {f(1)}')
```

    Função inicial avaliada nas aproximações iniciais:
    f(-1) = -0.45969769413186023
    f(0) = 1.0
    f(1) = 1.5403023058681398


#### Testando cada um dos métodos

Os métodos irão rodar por 9 iterações e será possível observar a qualidade da aproximação de cada um.

O método da secante, que é sujeito a erros por estar realizando divisão por 0, evita isso retornando os mesmos valores de `x` e `xo` que foram utilizados como entrada da função.


```python
for i in range(10):
    x_secante, xo_secante = cn.metodo_secante(f, x_secante, xo_secante)
    
    x_newton = cn.metodo_newton(f, flinha, x_newton)
    
    a_bi, b_bi = cn.metodo_bisseccao(f, a_bi, b_bi)


print('Raiz encontrada pelo método da secante:')
print(f'x = {x_secante} | xo ={xo_secante}')
print(f'f(x) = {f(x_secante)} | f(xo) = {f(xo_secante)}')

print('\nRaiz encontrada pelo método de Newton:')
print(f'x = {x_newton}')
print(f'f(x) = {f(x_newton)}')

print('\nIntervalo encontrado pelo método da bissecção:')
print(f'Raiz está entre: {(a_bi, b_bi)}')
print(f'f(a) = {f(a_bi)} | f(b) = {f(b_bi)}')
```

    Raiz encontrada pelo método da secante:
    x = -0.7390851332151607 | xo =-0.7390851332151607
    f(x) = 0.0 | f(xo) = 0.0
    
    Raiz encontrada pelo método de Newton:
    x = -0.7390851332151607
    f(x) = 0.0
    
    Intervalo encontrado pelo método da bissecção:
    Raiz está entre: (-0.7392578125, -0.73828125)
    f(a) = -0.0002890091467900868 | f(b) = 0.001345149751805108


## *aluno_exatas.circuitos_eletricos*

Esse módulo tem como função auxiliar o aluno que está cursando circuitos elétricos. Sua principal função é a implementação de funções que facilitam o uso de fasores.

#### Importando módulos úteis:


```python
import aluno_exatas.circuitos_eletricos as ce
import numpy as np
```

### Fasores

Essas funções permitem fazer a conversão entre números complexos nativos de Python e a representação fasorial (magnitude e defasagem).

#### Importando as funções relativas a fasores:


```python
from aluno_exatas.circuitos_eletricos.fasores import *
```

#### Definindo constantes para serem testadas

Vamos definir o mesmo número em duas representações diferentes, e verificar que as funções utilizadas retornam a mesma coisa.


```python
numero_complexo = 3 + 4j

magnitude = 5
angulo_rad = np.arctan(4 / 3)
angulo_deg = np.rad2deg(angulo_rad)

print(f'Ângulo em radianos: {angulo_rad:1.4f} e em graus: {angulo_deg:1.4f}')
```

    Ângulo em radianos: 0.9273 e em graus: 53.1301


#### Funções do módulo

O módulo define duas funções base:
* `complexo_para_polar`;
* `polar_para_complexo`.

No entanto, essas funções têm nomes muito longos, e portanto é mais simples utilizar as versões com nomes simplificados:
* `c2p`: converte números complexos para representação fasorial;
* `p2c`: converte a representação fasorial para um número complexo;
* `c2pr`: a mesma coisa que `c2p`, mas os ângulos são representados em radianos;
* `p2cr`: a mesma coisa que `p2c`, mas os ângulos são representados em radianos;


```python
print(f'O número {numero_complexo} em coordenadas polares utilizando graus é: {c2p(numero_complexo)}.')
print(f'O número {numero_complexo} em coordenadas polares utilizando radianos é: {c2pr(numero_complexo)}.\n')

print(f'O fasor de magnitude {magnitude} e com defasagem {angulo_deg:1.4f} graus ou {angulo_rad:1.4f} rad é igual a {p2c(magnitude, angulo_deg):1.4}')
```

    O número (3+4j) em coordenadas polares utilizando graus é: (5.0, 53.13010235415598).
    O número (3+4j) em coordenadas polares utilizando radianos é: (5.0, 0.9272952180016122).
    
    O fasor de magnitude 5 e com defasagem 53.1301 graus ou 0.9273 rad é igual a (3+4j)


#### Testando ida e volta das funções

Dado que as funções aqui exploradas permitem a conversão entre duas representações, é de se esperar que elas permitam reverter uma conversão. Isso pode ser observado abaixo.


```python
numero_complexo_novo = p2c(*c2p(numero_complexo))

print(f'Número complexo -  velho: {numero_complexo:1.4} e novo: {numero_complexo_novo:1.4}.\n')


magnitude_nova, angulo_deg_novo = c2p(p2c(magnitude, angulo_deg))

print(f'Magnitude - velha: {magnitude:1.4f} e nova: {magnitude_nova:1.4f}.')
print(f'Ângulo em graus - velho: {angulo_deg:1.4f} e novo: {angulo_deg_novo:1.4f}.\n')


magnitude_nova, angulo_rad_novo = c2pr(p2cr(magnitude, angulo_rad))

print(f'Magnitude - velha: {magnitude:1.4f} e nova: {magnitude_nova:1.4f}.')
print(f'Ângulo em radianos - velho: {angulo_rad:1.4f} e novo: {angulo_rad_novo:1.4f}.')
```

    Número complexo -  velho: (3+4j) e novo: (3+4j).
    
    Magnitude - velha: 5.0000 e nova: 5.0000.
    Ângulo em graus - velho: 53.1301 e novo: 53.1301.
    
    Magnitude - velha: 5.0000 e nova: 5.0000.
    Ângulo em radianos - velho: 0.9273 e novo: 0.9273.

