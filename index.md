## Trabalho 2 - Introdução
Esta atividade consiste na implementação de todas as etapas do pipeline gráfico anteriores à rasterização, última etapa do processo. Inicialmente os modelos são descritos no espaço do objeto, um espaço mais adequado em extensão e dimensões. Após sua descrição no espaço do objeto o modelo é submetido a várias transformações e após todas as transformações é levado ao espaço de tela. Logo após ser levado ao espaço de tela é feita a rasterização e projeção das primitivas na tela.

O pipeline gráfico consiste exatamente de uma sequência de passos necessários para transformar uma descrição geométrica/matemática de uma cena 3D em uma descrição visual na tela 2D. A imagem final é obtida por meio da rasterização das primitivas projetadas na tela. Basicamente, cada passo do pipeline gráfico consiste de uma transformação geométrica de um sistema de coordenadas (espaço) para outro, são eles :

![nice image](/icg/pipelinegrafico1.png)

## Desenvolvimento
A primeira etapa feita nesse trabalho foi criar um biblioteca de funções. As funções criadas foram as seguintes:

**multEscVet()**: que recebe um vetor e um escalar e retorna um vetor resultante da multiplicação daqueles;

**multMatVet()**: que recebe uma matriz e um vetor e retorna um vetor resultante da multiplicação daqueles;

**multMatMat()**: que recebe duas matrizes e retorna uma matriz resultante da multiplicação daquelas;

**subVetVet()**: que recebe dois vetores e retorna um terceiro vetor resultante da subtração daqueles;

**normaVet()**: que recebe um vetor e retorna um escalar como resultado da norma vetorial;

**produtoVetorial()**: que recebe dois vetores e retorna um terceiro vetor como resultado do produto vetorial;

### Espaço do objeto
É o espaço onde cada modelo é construído em seu próprio sistema de coordenadas, sendo o centro do objeto a origem do sistema. Nessa etapa são utilizados programas de modelagem, como o Blender, por exemplo. Nessa atividade foi utilizado o modelo da mascote do Blender  num arquivo obj.

![nice image2](/icg/a1.png)

Para carregar o modelo foi criado um vetor auxiliar e por meio de um loop for foi percorrido todos os vértices do objeto e copiado para esse vetor auxiliar, chamado de **v_obj**, por meio do qual foi possível iniciar a sequência do pipeline.

### Espaço do universo

Para levar todos os  objetos, que foram modelados em seus próprios sistemas de coordenadas, para o mesmo espaço, todos os vértices do espaço do objeto são transformados e levados ao espaço do universo.

Para isso, é usada uma matriz de modelagem, a matriz **model**, que contém uma série de transformações geométricas que podem ser: rotação, translação, escala e shear. Assim, os vértices do espaço do objeto são multiplicados pela matriz model e são levados para o espaço do universo.

Nesse trabalho foram criadas as seguintes funções:

**escala()**: cria uma matriz de escala a partir dos parâmetros de entrada sx, sy, e sz e multiplica pela model (inicialmente será a identidade);

**translação()**: cria uma matriz de translação a partir dos parâmetros de entrada dx, dy, e dz e multiplica pela model (inicialmente será a identidade);

**rotação()**: cria uma matriz de rotação em x, y ou z de acordo com a escolha informada como parâmetro e multiplica pela model (inicialmente será a identidade);

**shear()**: cria uma matriz de shear e multiplica pela model (inicialmente será a identidade);

### Espaço da Câmera

Nessa etapa os vértices  do espaço do universo são transformados para o espaço da câmera.  Isso é feito por meio da multiplicação desses vértices pela matriz View que possui uma translação e uma rotação.

#### O sistema de coordenadas da câmera
Para efetuar a transformação do espaço do universo para o da câmera é necessário realizar uma mudança de base, pois o sistema de coordenadas da câmera possui 3 propriedades:

Independência Linear;

Ortogonalidade;

Base ortonormal.

Para construir o sistema de coordenadas da câmera inicialmente foi dado como entrada os 3 parâmetros da câmera:

Posição da câmera: p = (px , py , pz);

Ponto para onde a câmera está olhando (lookat) : l = (lx, ly, lz);

Vetor up da câmera no espaço do universo: u = (ux, uy, uz).

Usando a posição da câmera e o vetor lookat é possível calcular a direção de visão da câmera d = (dx, dy, dz) , calculando: d = p – l.

A partir desses dados foi possível calcular o x, y e z da câmera.

#### Construindo a matriz View

Como dito anteriormente, a matriz View é formada por uma rotação e uma translação. A matriz de rotação é formada pelos vetores x, y e z da câmera e a matriz de translação é formada pelo vetor de posição da câmera. Como descrito abaixo:

![nice image 3](/icg/a2.png)

No código foi criada a função sistOrtCam() que recebe como entrada os vetores de parâmetros da câmera e fornece como saída as coordenadas x, y e z da câmera. Depois essas coordenadas e o vetor de posição são passados à função constMview() que constrói a matriz View. Finalmente, a matriz model foi multiplicada pela view criando a matriz model view.

### Espaço de recorte
É o espaço onde ocorrerá o recorte de todos os vértices que não estão na visão da câmera para que estes não percorram o pipeline gráfico, já que não precisam aparecer na tela. E é nessa etapa que os vértices  do espaço da câmera são transformados para o espaço de recorte.

Esse processo é feito por meio da multiplicação desses vértices pela matriz de projeção que simula a distorção perspectiva, ou seja, os objetos mais próximo parecem maiores enquanto os mais afastados parecem menores.

#### Construindo a matriz de projeção
Um ponto no espaço da câmera é projetado sobre a view plane e por meio de semelhança de triângulos é possível encontrar as coordenadas x, y e z do espaço de recorte.

![nice image 4](/icg/a3.png)

Após calcular os valores das coordenadas de x, y e z do espaço de recorte foi possível criar a matriz de projeção Mp. Além disso, também é possível observar que a câmera se encontra deslocada em d unidades até a origem, então é preciso fazer uma translação de d unidades para que ela fique na origem, matriz Mt, com o novo sistema de coordenada e essa translação é possível construir a matriz de projeção Mpt:

![nice image 5](/icg/a4.png)

Algo muito interessante de se notar no sistema de recorte é que a coordenada homogênea normalmente é diferente de 1.

Finalmente, a matriz model view é multiplicada pela matriz de projeção formando a matriz model view projection. Logo após os vértices do espaço do objeto são multiplicados por essa matriz levando-os direto para o espaço de recorte.

### Espaço canônico
O espaço canônico é um espaço formado por um cubo de coordenadas unitárias onde está contida toda a cena. Seria interessante que após homogeneizar os vértices no espaço de recorte fossem levados ao espaço canônico, porém a matriz Mpt é muito simples não sendo possível. Dessa forma a definição do espaço  canônico é feito em duas etapas:

Homogeneização, dividindo os vértices no espaço de recorte pela coordenada W;

Multiplicação por uma matriz que contém escalas e translações e que define o cubo de tamanho unitário

Apesar dos vértices não estarem no verdadeiro espaço canônico, ocorrerá a distorção perspectiva e, nesse trabalho, será considerado que está no espaço canônico. Nesse trabalho, porém foi efetuada apenas a divisão por W dos vértices do espaço de recorte ou seja os vetores no espaço de recorte são divididos por sua quarta coordenada levando-os ao espaço canônico.

### Espaço de tela
No trabalho 1, a rasterização era feita diretamente no espaço de tela, porém esse é um estágio posterior do pipeline gráfico. Nessa etapa ao receber os vértices no espaço canônico essas coordenadas são transformadas para o espaço de tela para só depois ser feita a rasterização.

![nice image 6](/icg/a5.png)

### Construção da matriz view port
A matriz view port construída nesse trabalho é composta por 3 transformações  duas escalas e uma traslação. Detalhadas abaixo.

#### Espelhamento em y
Nessa etapa é necessário que o y seja espelhado porque no espaço canônico o y cresce para cima e no espaço de tela para baixo.

#### Translação em x e y
Como no espaço canônico os valores das coordenadas x e y iniciam em -1 e no espaço de tela iniciam em 0 é necessário efetuar uma translação, com fator igual a 1 tanto em x quanto em y. Dessa forma, os valores das coordenadas passam a iniciar em 0.

#### Escala ao longo de x e y
A terceira e ultima transformação utilizada é uma escala em x e em y, pois os valores das coordenadas x e y estão indo de 0 a 2 e precisam ir de 0 até a largura da tela em x e de 0 até a altura da tela em y. Assim o fator de translação utilizado é (w-1)/2 em x e (h-1)/2 em y, sendo w e h a largura e a altura da tela, respectivamente.

Nesse trabalho a função criada para realizar a construção da matriz view port foi a constViewPort(). Observe todas as transformações aplicadas para formar a matriz view port.

![nice image 7](/icg/a6.png)

Enfim, os vértices do espaço do espaço canônico foram multiplicados pela matriz view port. Porém, como os vértices no espaço de tela são inteiros ainda foi necessário efetuar um arredondamento das suas coordenadas. E assim os vértices do espaço canônico foram levados ao espaço de tela.

### Desenhando na tela
Para finalizar foi percorrida a lista de faces já transformadas por meio de um laço for chamando a função drawTriangle(), desenvolvida np trabalho 1, para rasterizar os triângulos do objeto agora com os vértices em x e y inteiros e com distorção perspectiva.

### Resultados
Os parâmetros posiçao, lookat e up da câmera utilizados foram sempre os mesmos, mudando apenas a distância d da câmera até o near plane:

Posição da câmera = (0, 0, 4);

Lookat = (0, 0, 0);

Vetor up = (0, 1, 0);

![nice image 8](/icg/a7.png)

![nice image 9](/icg/a8.png)

#### Rotacionando a mascote
Para efetuar a rotação nos eixos x, y, z da mascote, foram utilizadas suas respectivas matrizes de rotação

![nice image 10](/icg/a9.png)

![nice image 11](/icg/a10.png)

![nice image 12](/icg/a11.png)

No vídeo é possível observar a rotação da mascote sendo efetuada nos três eixos ao redor da origem

![nice image 13](/icg/a12.png)

Confira o vídeo [aqui](https://youtu.be/fVnaGKCfnbg)

### Dificuldades
Por mais incrível que possa parecer a maior dificuldade foi em implementar a rotação da mascote, pois inicialmente a matriz model estava sendo iniciada com a identidade e os valores de profundidade estavam sendo perdidos  na rotação apesar de ter seguido o pipeline corretamente. Além disso, houveram outras dificuldade que foram contornadas mais facilmente, como falhas de segmentação, desenho fora do ângulo de visão, etc.

###  Referências Bibliográficas
Foley et al.; Computer Graphics – Principles and Practice with C; Addison-Wesley.

Slides do Profº Christian Pagot utilizados em aula

## Trabalho 1 - Introdução
Esse artigo constitui o relatório do primeiro projeto da disciplina de ICG, ministrada pelo Prof. Christian Pagot.

A atividade propõe a criação de quatro funções: PutPixel, DrawLine, DrawTriangle e DrawFilledTriangle. As funções são baseadas em algoritmos de rasterização feita através de um framework disponibilizado pelo próprio professor, que simula o acesso a memória de vídeo.

Para representar cores em um computador, são utilizados 8 bits para cada componente do pixel, dando 256 níveis de intensidade para cada cor. Combinando os 3 canais (RGB) é possível representar mais ou menos 16 milhões de cores. Além disso, existe um canal A (alpha) que diz respeito a transparência do pixel. Usaremos então o espaço RGBA.

Sendo assim, considerando que pixels são compostos por 4 canais, cada um com 8 bits de memória, temos então 32 bits de informação por pixel. Só nos resta então pintar os pixels na tela, pra isso basta escrever na memória de vídeo os valores de cada pixel. Porém nos computadores modernos, o acesso à memória de vídeo é restrito, e não é possível alterá-los facilmente. Por isso foi concedido o framework para realização desse trabalho.

![useful image](/icg/rgba_text_logo@2x.png)

## Função PutPixel

A função recebe um objeto da classe vetor e outro da classe cor, cada elemento das classes compõem um pixel que é rasterizado através do ponteiro FBptr que aponta para o pixel (0,0) do framebuffer. O pixel possui 4 componentes RGBA que criam sua cor.

```markdown

void PutPixel(vector v, color c){
    int coordinates = 4*v.x + 4*v.y*IMAGE_WIDTH;

    FBptr[coordinates + 0] = c.r;
    FBptr[coordinates + 1] = c.g;
    FBptr[coordinates + 2] = c.b;
    FBptr[coordinates + 3] = c.a;
}

```

podemos adicionar um tratamento para caso as entradas ultrapassarem as bordas da tela :

```markdown

if(v.x < 0 || v.y < 0 || v.x > IMAGE_WIDTH || v.y > IMAGE_HEIGHT) return;

```

### Função DrawLine

A função utiliza o Algoritmo de Bresenham para rasterizar uma linha na tela, recebe dois vetores, um inicial e um final, e a cor de cada pixel.

![useful image 2](/icg/screenshot-from-2017-09-10-22-46-05-e1505095713992.png)

O algoritmo básico de Bresenham funciona apenas no primeiro octante, para que funcione em todos precisamos manipular as variáveis de acordo com as propriedades observadas em cada octante:

![useful image 3](/icg/octantesreal.gif)

Outra limitação é que temos que escrever a linha da esquerda para a direita, isso pode ser facilmente manipulado caso o pixel inicial venha depois do final (dx < 0), simplesmente fazendo uma chamada recursiva com as posições trocadas:

```markdown

if(dx < 0){ DrawLine(v1,v0,c2,c1); return; }

```

Após manipulação das variáveis para seus respectivos octantes obtemos o resultado final:

![useful image 4](/icg/screenshot-from-2017-09-10-21-42-16-e1505099482973.png)

## Interpolação de Cores

A interpolação de cores pode ser implementada modificando a porcentagem da cor RGBA inicial e a porcentagem da cor RGBA final, resultando numa nova e substituindo-a em cada pixel escrito por vez ao longo da linha, como observado aqui. A porcentagem é calculada dividindo a distância total da linha pela distância parcial do pixel a ser escrito.

```markdown

distT = Distance(v0, v1);
distP = Distance(v, v1);

percent = distT/distP;

newColor.r = c1.r * percent + (1-percent) * c2.r;
newColor.g = c1.g * percent + (1-percent) * c2.g;
newColor.b = c1.b * percent + (1-percent) * c2.b;
newColor.a = c1.a * percent + (1-percent) * c2.a;

```

Por exemplo, uma linha que inicia azul e termina verde tem a cor do primeiro pixel como:

```markdown

newColor = azul * 1.00 + verde * 0.00

```

A cor do pixel do meio da linha como:

```markdown

newColor = azul * 0.50 + verde * 0.50

```

E a cor do pixel final como:

```markdown

newColor = azul * 0.00 + verde * 1.00

```

A função Distance foi criada para facilitar o calculo das distâncias entre pixels, dado por:

![useful image 5](/icg/cace1-distt.gif)

## Função DrawTriangle

A função recebe como parâmetro três vetores, e suas respectivas cores, a função cria um triângulo chamando a função DrawLine de um vetor a outro, formando um triângulo

```markdown

void DrawTriangle(vector v1, vector v2, vector v3, color c1, color c2, color c3){
     DrawLine(v1, v2, c1, c2);
     DrawLine(v2, v3, c2, c3);
     DrawLine(v3, v1, c3, c1);
}

```

Resultando em:

![useful image 6](/icg/screenshot-from-2017-09-10-22-25-42-e1505101771971.png)

## Função DrawFilledTriangle

A função cria um triângulo preenchido, recebendo como parâmetro três vetores e suas respectivas cores. A função se trata da repetição da função DrawLine, porém ao escrever cada pixel, escreve uma linha que vai do pixel atual ao terceiro vetor do parâmetro.

![useful image 7](/icg/screenshot-from-2017-09-10-22-32-30-e1505102073327.png)

Podemos observar falhas em alguns pixels, eles podem ser preenchidos ao chamar a função algumas vezes, em diferentes combinações:

```markdown

DrawFilledTriangle(v1, v2, v3, red, blue, green);
 DrawFilledTriangle(v3, v1, v2, green, red, blue);
 DrawFilledTriangle(v2, v3, v1, blue, green, red);
 
 ```
 
 Também é possível criar uma função que simplesmente chama a função DrawFilledTriangle várias vezes, alternando os parâmetros de posição.
 
## Referências
 
 [Drawing Line Using Bresenham Algorithm](http://tech-algorithm.com/articles/drawing-line-using-bresenham-algorithm/)
 
 [Generate n colors between two colors](https://stackoverflow.com/questions/17544157/generate-n-colors-between-two-colors)
 
 [HexColorTool](https://www.hexcolortool.com/)
 
 Slides do Profº Christian Pagot utilizados em aula
