## Introdução
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
