## Introdução
Esse artigo constitui o relatório do primeiro projeto da disciplina de ICG, ministrada pelo Prof. Christian Pagot.

A atividade propõe a criação de quatro funções: PutPixel, DrawLine, DrawTriangle e DrawFilledTriangle. As funções são baseadas em algoritmos de rasterização feita através de um framework disponibilizado pelo próprio professor, que simula o acesso a memória de vídeo.

Para representar cores em um computador, são utilizados 8 bits para cada componente do pixel, dando 256 níveis de intensidade para cada cor. Combinando os 3 canais (RGB) é possível representar mais ou menos 16 milhões de cores. Além disso, existe um canal A (alpha) que diz respeito a transparência do pixel. Usaremos então o espaço RGBA.

Sendo assim, considerando que pixels são compostos por 4 canais, cada um com 8 bits de memória, temos então 32 bits de informação por pixel. Só nos resta então pintar os pixels na tela, pra isso basta escrever na memória de vídeo os valores de cada pixel. Porém nos computadores modernos, o acesso à memória de vídeo é restrito, e não é possível alterá-los facilmente. Por isso foi concedido o framework para realização desse trabalho.

![useful image](/icg/rgba_text_logo@2x.png)
