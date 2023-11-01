# Snake-Game-ARDUINO

> Projeto multidisciplinar do 1º semestre de ADS na FATEC Guarulhos.
> 
> Prof. RODRIGO AMORIM MOTTA CARVALHO (Laboratório de Hardware)
> 
> Prof.ª JANE MARIA DOS SANTOS EBERSON (Algoritmos e Lógica de Programação)


# [Jogue e simule o projeto aqui](https://www.tinkercad.com/things/hyu77o4bxZS?sharecode=uqQpXzAwfeA78uUUr-lCLAQy2km67BkepBK9EkrpN24)

# Informações
 
 Tinkercad | Arduino | C
-----|-----|----

Inicio Projeto | Fim Projeto
-|-
24/10/2023 | 30/10/2023


# Screenshots

>![image](https://github.com/luizfernandope/Snake-Game-ARDUINO/assets/74561722/def8b7e0-2f91-4e6d-a687-bd1fc1631f3a)

# Código
````C
#include <Adafruit_NeoPixel.h>
#include <LiquidCrystal.h>

#define PIN 2	 // input pin Neopixel is attached to
#define NUMPIXELS      64 // number of neopixels in strip
#define pinCima 4
#define pinBaixo 7
#define pinDireita 6
#define pinEsquerda 5
Adafruit_NeoPixel leds = Adafruit_NeoPixel(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
LiquidCrystal lcd(13,12,11,10,9,8);

int ledsLinha = 8, qtdLinhas = 7;//total linhas - 1(para facilitar calculos)
int proximo = 0;
bool cima = false, baixo = false, direita = false, esquerda = false;
int posicaoMaca = random(0, NUMPIXELS-1);;
int tamanhoCobra = 0, cobra[64];
bool perdeu = false;
int lag = 700;//tempo de delay (velocidade do jogo)

void zerarMove(){
 cima = false;
 baixo = false;
 direita = false;
 esquerda = false;
}


void movimentacao(){
  if(cima)
  {
    proximo = proximo - ledsLinha;
    if(proximo<0)
      proximo = (proximo+8) + (8 * qtdLinhas );
  }
  else if (baixo)
  {
    proximo = proximo + ledsLinha;
    if(proximo>63)
    {
      int teste = (proximo-8) - (8 * qtdLinhas );
      if (proximo>=0)
        proximo = teste;
    }
  }
  else if(direita)
  {
    proximo++;
    for(int i=1;i<=qtdLinhas+1;i++){
      if(proximo == i*8)
        proximo = proximo - ledsLinha;
    }
  }
  else if(esquerda)
  {
    proximo--;
    for(int i=1;i<=qtdLinhas+1;i++)
    {
      if(proximo == i*7+(i-1) || proximo <0)
      {
        proximo = proximo + (ledsLinha);
        break;
      }
    }
  }
  
  for(int i = tamanhoCobra;i>=1;i--)//for para fazer o corpo
  {
    cobra[i] = cobra[i-1];//ultimo led da cobra recebe o posterior
  }
  
  cobra[0] = proximo;//colcando a cabeça da cobra a diante
  
  leds.fill(0,0,0);//falando para apagar todos pixels
  leds.show();//apagando todos pixels
  
  for(int i=0;i<=tamanhoCobra;i++)
  {
    leds.setPixelColor(cobra[i], leds.Color(0, 230, 0));//led verde do corpo da cobra
    if(i==0)
      leds.setPixelColor(cobra[i], leds.Color(222,255,0));//led amarelo da cabeça da cobra
  }
  leds.show();//mostrando os leds da cobra
}



void setup() {
  leds.begin();//iniciando os leds (faixas NeoPixel)
  lcd.begin(16,2);//iniciando o display lcd
  pinMode(pinCima, INPUT);//colocando o pino em modo de leitura
  pinMode(pinBaixo, INPUT);//colocando o pino em modo de leitura
  pinMode(pinDireita, INPUT);//colocando o pino em modo de leitura
  pinMode(pinEsquerda, INPUT);//colocando o pino em modo de leitura
}

void loop() {
  
if(!perdeu)//se não perdeu
{
  lcd.setCursor(2,0);//colocando o ponteiro do lcd na coluna/linha desejada
  lcd.print("Jogo da cobra");//exibindo no lcd o texto desejado
  lcd.setCursor(2,1);//colocando o ponteiro do lcd na coluna/linha desejada
  lcd.print("Pontuacao = ");//exibindo no lcd o texto desejado
  lcd.print(tamanhoCobra+1);//exibindo no lcd o texto desejado
  if(digitalRead(pinCima)==1 && !baixo)
  {
    zerarMove();
    cima = true;
  }
  else if(digitalRead(pinBaixo)==1 && !cima)
  {
    zerarMove();
    baixo = true;
  }
  else if(digitalRead(pinDireita)==1 && !esquerda)
  {
    zerarMove();
    direita = true;
  }
  else if(digitalRead(pinEsquerda)==1 && !direita)
  {
    zerarMove();
    esquerda = true;
  }
  
  movimentacao();
  colidiu();
  maca();
  delay(lag);
} 
}

void maca(){
  if(cobra[0]==posicaoMaca)
  {
    tamanhoCobra++;
    cobra[tamanhoCobra] = cobra[tamanhoCobra-1];
    posicaoMaca = random(0, NUMPIXELS-1);
    bool macaOK = false;//variavel logica para ver se posicao da maçã esta ok 
    while(macaOK == false)//enquanto posicao maçã for irregular
    {
      posicaoMaca = random(0, NUMPIXELS-1);//spawnando maça
      for(int i=0;i<=tamanhoCobra;i++)//for para checagem da posicao da maçã
      {
      	if(cobra[i] == posicaoMaca)//se cobra[i](qual parte da cobra pelo loop) for igual a posicao da maça
          posicaoMaca = random(0, NUMPIXELS-1);//respawna maça em outro luga
        else if(i==tamanhoCobra)//se maçã nao nasceu dentro da cobra
          macaOK = true;//fim loop
      }
    }
    
    if(tamanhoCobra%3==0)//a cada pontuação multipla de 3
  	{
    	lag = lag-200;//aumenta a velocidade do jogo
    	if(lag<100)
     	 lag = 80;// velocidade mais rápida do jogo
  	}
  }
  leds.setPixelColor(posicaoMaca, leds.Color(250, 0, 0));//colocando a cor vermelha da maça
  leds.show();//acendendo led da maça
}

void colidiu()
{
  for(int v=tamanhoCobra;v>0;v--)//for para verificar colisoes da cobra em seu corpo
    {
      if(cobra[0] == cobra[v])//se cabeça da cobra estiver na mesma posicao de algum led do seu corpo
      {
        zerarMove();//para de se mexer
        perdeu = true;
        leds.setPixelColor(cobra[0], leds.Color(7,7,7));//cabeça da cobra fica cinza
    	leds.show();//acende a cabeça da cobra 
        lcd.clear();//limpa o display lcd
        lcd.setCursor(2,0);//colocando o ponteiro do lcd na coluna/linha desejada
  		lcd.print("voce perdeu");//exibindo no lcd o texto desejado
  		lcd.setCursor(2,1);//colocando o ponteiro do lcd na coluna/linha desejada
  		lcd.print("Pontuacao = ");//exibindo no lcd o texto desejado
  		lcd.print(tamanhoCobra+1);//exibindo no lcd o texto desejado
      }
    }
}

/* Esquema dos leds
0,1,2,3,4,5,6,7,
8,9,10,11,12,13,14,15,
16,17,18,19,20,21,22,23,
24,25,26,27,28,29,30,31,
32,33,34,35,36,37,38,39,
40,41,42,43,44,45,46,47,
48,49,50,51,52,53,54,55,
56,57,58,59,60,61,62,63
*/
````

# Autores

* Luiz Fernando Pegorari
* Ester Paixão de Freitas
* Leonardo Durval di Bonito
* Lucas Silva Contieri
