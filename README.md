# Juego-de-la-brisca-Bristle-game
//Escribimos las bibliotecas necesarias
#include <iostream>
#include <cstdlib>
#include <ctime>
#include <sstream>
#include <fstream>

using namespace std;

//Declaramos constantes
const int NUM_CARTAS_JUGADOR=3;
const int NUM_PALOS=4;
const int As=1;
const int Sota=8;
const int Caballo=9;
const int Rey=10;
//PRACTICA 2
const int NUM_RONDAS=3;
const int NUM_TOTAL_CARTAS=40;

//Declaramos los tipos
typedef enum {oros, espadas, copas, bastos} tPalo; //Declaramos el tipo enumerado

typedef struct {
    tPalo palo;
    int num;
}tCarta;

typedef tCarta tCartasJugador[NUM_CARTAS_JUGADOR];

typedef tCarta tArrayCartasMazo[NUM_TOTAL_CARTAS];
typedef struct {
    tArrayCartasMazo cartas;
    int contador;      //contador
}tMazo;

//PROTOTIPOS
//Devuelve una cadena que representa una carta
string cadPalo(tPalo palo);
//Devuelve un palo aleatorio
tPalo paloAleatorio();
//Devuelve el número de puntos que vale la carta
int valorCarta(tCarta carta);
//Devueleve qué carta es la ganadora
int indicarCartaGanadora (tCarta carta0, tCarta carta1, tPalo triunfo);
//Crea tres cartas aleatorias y las incluye en el array de las cartas del jugador pasado por parámetro
void repartirCartas(tMazo & mazo, tCartasJugador cartasJugador);
//Muestra por consola las cartas que hay en cartasJugador
void mostrarCartas(tCartasJugador cartasJugador);
//Pide al usuario que elija una carta
tCarta  pedirCarta(tCartasJugador cartasJugador);
// Devuelve  una  carta  elegida aleatoriamente de las tres cartas generadas anteriormente y reponer la carta elegida por el  usuario por  una carta obtenida del mazo
tCarta elegirMaquina(tCartasJugador &cartasJugador, tMazo &mazo);
//Función que realiza el juego en esta nueva versión de acuerdo a las reglas de la brisca
void jugarV2();
//Crea un mazo con todas las cartas estando ordenado
void  crearMazo(tMazo  &mazoOrdenado);
//Coge cartas aleatoriamente del mazo pasado por parámetro para crear un nuevo mazo barajado
void barajarMazo(tMazo &mazoOrdenado, tMazo & mazo);
//Cadena de cartas
string cadCarta(tCarta carta);
//Guarda los resultados de una partida en un archivo “resultado.txt”
void guardarResultado(int puntosUsuario, int puntosMaquina, string nombreUsuario);
//Obtiene la carta en la última posición ocupada de mazo y disminuye el contador correspondiente
tCarta obtenerCarta(tMazo & mazo);


//Función principal
int main(){

    //Semilla que hace que se comporte de manera aleatoria las cartas
    srand(time(NULL));
    //Llamamps a la función que realiza el juego
    jugarV2();

 return 0;
}

//Devuelve la cadena de caracteres que representa el palo pasado por parámetro
string cadPalo(tPalo palo){
        string cad;

        if (palo == oros){
            cad = "oros";
        }
        else if (palo == espadas){
            cad = "espadas";
        }

        else if (palo == copas){
            cad = "copas";
        }
        else if (palo == bastos){
            cad = "bastos";
        }
        return cad; //Cerramos cadena
}

//Devuelve un palo aleatorio
tPalo paloAleatorio(){
    int nPalo = rand()%NUM_PALOS;
    tPalo palo = (tPalo)nPalo;
    return palo;
}


//Escribimos cada caso para cada carta
string cadCarta(tCarta carta){
    stringstream flujo;
    switch (carta.num){
    case 1 :
        flujo<<"As";
        break;
    case 8 :
        flujo<<"Sota";
        break;
    case 9 :
        flujo<<"Caballo";
        break;
    case 10 :
        flujo<<"Rey";
        break;
    default :
        flujo<<carta.num;

    }
        flujo<<" de "<<cadPalo(carta.palo);
        return flujo.str();
}

//Devuelve el número de puntos que vale la carta
int valorCarta(tCarta carta){
    int valor;
        switch(carta.num){
        case 1 : valor=11;                  //Caso para el As
    break;
        case 3 : valor=10;                  //Caso para el Tres
    break;
        case 10: valor=4;                   //Caso para el Rey
    break;
        case 9 : valor=3;                   //Caso para el Caballo
    break;
        case 8 : valor=2;                   //Caso para la Sota
    break;
        default : valor=0;                  //Caso para el resto de cartas que no sean las anteriores
    break;
    }
return valor;

}

//Indicamos qué carta ha resultado ganadora de acuerdo a las reglas del juego
int indicarCartaGanadora (tCarta carta0, tCarta carta1, tPalo triunfo){
    bool gana0;
    int ganadora;
    //Mismos palos
    if(carta0.palo==carta1.palo){
        if(valorCarta(carta0)!=valorCarta(carta1))
            gana0=valorCarta(carta0)>=valorCarta(carta1);
        else
            gana0=(carta0.num>=carta1.num);
    //Diferentes palos
    }else{
        gana0=(carta1.palo!=triunfo);
    }
    if(gana0)                                           //Gana el usuario
        ganadora=0;
    else
        ganadora=1;                                     //Gana la máquina
    return ganadora;

}
//Crea tres cartas aleatorias y las incluye en el array de las cartas del jugador pasado por parámetro
void repartirCartas(tMazo & mazo, tCartasJugador cartasJugador){
    for (int n=0; n<NUM_CARTAS_JUGADOR; n++){
        cartasJugador[n]=obtenerCarta(mazo);
    }
}
//Muestra las cartas que hay en cartasJugador
void mostrarCartas(tCartasJugador cartasJugador){
    cout<<"Tus cartas son:";
    for(int n=0; n<NUM_CARTAS_JUGADOR; n++){
        cout<<" ("<<n+1<<") "<<cadCarta(cartasJugador[n]);
    }
cout<<endl;

}
//Pide al usuario que elija una carta y devuelve la carta que ocupa dicha posición
tCarta  pedirCarta(tCartasJugador &cartasJugador,  tMazo &mazo){
    int num;
        cout<<"Elige una carta (indica la posicion de la carta - entre 1 y 3 -):"<<endl;
        cin>>num;
    //Mientras no se introduzca un número correcto, se pedirá de nuevo
    while(num<1 ||num>3){
        cout <<"Elige una carta (indica la posicion de la carta - entre 1 y 3 -):"<<endl;
        cin>>num;}
    num--;
    //Repone la carta elegida por el usuario por una del mazo llamando a obtener carta.
    tCarta carta=cartasJugador[num];
    cartasJugador[num]=obtenerCarta(mazo);
    return carta;
    }

//Devuelve una carta elegida aleatoriamente de las tres pasadas por parámetro en cartasJugador
tCarta elegirMaquina(tCartasJugador &cartasJugador, tMazo &mazo){
    int posAleatoria=rand()%NUM_CARTAS_JUGADOR;
    cartasJugador[posAleatoria];
    tCarta carta=cartasJugador[posAleatoria];
    cartasJugador[posAleatoria]=obtenerCarta(mazo);
    return carta;
}

    //PRÁCTICA 2
//Crea un mazo con todas las cartas estando ordenado
void  crearMazo(tMazo  &mazoOrdenado){
    tCarta carta;
    mazoOrdenado.contador=0;
        for(int i=0; i<NUM_PALOS; i++){
            for(int j=As; j<=Rey; j++){
                carta.palo = (tPalo)i;
                carta.num = j;
                mazoOrdenado.cartas[mazoOrdenado.contador]=carta;           //Ponemos la carta creada en el mazo
                mazoOrdenado.contador++;
        }
    }
}

//Coge cartas aleatoriamente del mazo pasado por parámetro para crear un nuevo mazo barajado
void barajarMazo(tMazo &mazoOrdenado, tMazo & mazo){
    mazo.contador=0;
    tCarta carta;
    tCarta ultima;
    while(mazoOrdenado.contador>0){
       int carta_aleatoria = rand()%mazoOrdenado.contador;
       carta = mazoOrdenado.cartas[carta_aleatoria];
       mazo.cartas[mazo.contador]=carta;
       mazo.contador++;
       ultima = mazoOrdenado.cartas[mazoOrdenado.contador-1];
       mazoOrdenado.cartas[carta_aleatoria]=ultima;
       mazoOrdenado.contador--;
    }
}

//Obtiene la carta en la última posición ocupada de mazo y disminuye el contador correspondiente
tCarta obtenerCarta(tMazo & mazo){
    tCarta carta = mazo.cartas[mazo.contador-1];
    mazo.contador--;
    return carta;
}

//Guarda los resultados de una partida en un archivo “resultado.txt”
void guardarResultado(int puntosUsuario, int puntosMaquina, string nombreUsuario){
    ofstream archivo;
    archivo.open("resultado.txt");                                                  //Abrimos el archivo
    archivo<<nombreUsuario<<" "<<puntosUsuario<<" "<<"puntos, maquina "<<puntosMaquina<<" puntos"<<endl;
    archivo.close();                                                                //Cerramos el archivo
}

//Función que realiza el juego
void jugarV2(){
    tMazo mazoOrdenado, mazo;
    //Comienzo del juego
    cout<<"\n\t   BIENVENIDO AL JUEGO DE LA BRISCA"<<endl;
    cout<<"\t--------------------------------------\n"<<endl;
    tCarta carta;
    //1.Se elige aleatoriamente un palo como triunfo y se muestra por pantalla
    tPalo triunfo=paloAleatorio();

    //Lo mostramos por pantalla
    cout<<"El palo triunfo es "<<cadPalo(triunfo)<<".\n"<<endl;

    //2.El programa inicializará los contadores de puntos de cada jugador a cero
    int puntosJugador=0, puntosMaquina=0;

    //3.El programa creará un mazo con todas las cartas
    crearMazo(mazoOrdenado);

    //4.El programa obtendrá un nuevo mazo barajeado a partir del anterior mazo
    barajarMazo(mazoOrdenado, mazo);

    //5.El programa asignará tres cartas del mazo a cada jugador
    tCartasJugador cartasUsuario;
    tCartasJugador cartasMaquina;
    repartirCartas(mazo, cartasUsuario);
    repartirCartas(mazo, cartasMaquina);

    //11.Se volverá al paso 6 si no se ha alcanzado el NUM_RONDAS, creando un bucle hasta completar el número de rondas.
    for(int i=0; i<NUM_RONDAS; i++){
            cout<<"\t----------------RONDA "<<i+1<<"---------------\n"<<endl;
    //6.El  jugador máquina escogerá  aleatoriamente  una  carta  y  la mostrará  por  pantalla. Repondrá la carta echada obteniendo una del mazo.
        tCarta cartaMaquina = elegirMaquina(cartasMaquina, mazo);

        cout<<"La maquina echa: "<<cadCarta(cartaMaquina)<<"."<<endl;

    //7-8.Se mostrarán las tres cartas del usuario y el programa pedirá al usuario que escoja una. Repondrá la carta echada obteniendo una del mazo.
        mostrarCartas(cartasUsuario);
        tCarta cartaUsuario;
        cartaUsuario=pedirCarta(cartasUsuario, mazo);

    //9-10.El programa aplicará las reglas de la Brisca y mostrará por pantalla quién es el ganador y cuántos puntos ha ganado.
        int ganador, sumatorio;
        ganador=indicarCartaGanadora(cartaMaquina, cartaUsuario, triunfo);
        sumatorio=valorCarta(cartaMaquina)+ valorCarta(cartaUsuario);
        if(ganador==0){
              cout<<"Ha ganado la maquina con "<<sumatorio<<" puntos.\n"<<"\nLo siento, vuelve a intentarlo.\n"<<endl;

              puntosMaquina = puntosMaquina + sumatorio;
        }else{
              cout<<"Ha ganado el usuario con "<<sumatorio<<" puntos.\n"<<"\nEnhorabuena, has ganado!\n"<<endl;
              puntosJugador= puntosJugador + sumatorio;
        }
    }

    //12.Se pide al usuario su nombre y se guardan los resultados
    cout<<"\t-----------FIN DE LA PARTIDA-----------\n"<<endl;
    cout<<"\tGracias por jugar, hasta la proxima!\n"<<endl;
    cout<<"Indique su nombre para guardar el resultado de la partida: "<<endl;
    string nombreUsuario;
    cin>>nombreUsuario;
    guardarResultado(puntosJugador, puntosMaquina, nombreUsuario);
}
