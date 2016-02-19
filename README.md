# bomberman
//animacion
#include "animacion.hpp"

Animacion::Animacion(SDL_Surface * img_grilla,int filas,int columnas,char * frames,int x,int y,int id):
    control_fra(frames){

    this->imgGrilla=img_grilla;
    this->x=rect.x=x;
    this->y=rect.y=y;
    rect.w=rect.h=16;
    self_kill=false;
    type=ANIMACION;
    this->id=id; 
    f=filas;
    c=columnas;  
    delay=0;
    loop=0;
    indexInicioAniEnd=0;
}

void Animacion::update(Uint8 * teclas){

	if (++delay > DELAY_CAMBIO){
        if(control_fra.avanzar()){
            if(loop!=-1&&--loop<0)
                disable();
            else if(indexInicioAniEnd!=0){
                control_fra.setInicioAnimacion(indexInicioAniEnd);
                indexInicioAniEnd=0;
                control_fra.reiniciar();
            }
        }
        delay=0;
	}
}
void Animacion::disable(){
    kill();
}

void Animacion::setCuadrosFrames(char * frames){
    control_fra.setCuadros(frames);
}

void Animacion::draw(SDL_Surface * screen){
    if(x+16>0&&x<W_SCREEN&&y<H_SCREEN&&y+16>0)
        imprimir_desde_grilla (imgGrilla,control_fra.cuadro(), screen, x,y,f,c,0);
}


// bloque 

#include "bloque.hpp"

Bloque::Bloque(Juego * juego,int x,int y,int id):
    Animacion(NULL,0,0,"0,0,0,0,0,1,1,1,1,2,2,2,2,3,3,3,4,4,5",x,y,id){
    this->juego=juego;
    idTileBloque=juego->getIdTile();
    type=BLOQUE;
}

void Bloque::disable(){
    if(juego->isBloqueItem(x,y)){
        juego->addSprite(ITEM,x,y,juego->getTipoNuevoItem(true));
    }
    juego->romperBloque(x,y);
    kill();
}

void Bloque::draw(SDL_Surface * screen){
    if(x+16>0&&x<W_SCREEN&&y<H_SCREEN&&y+16>0)
        imprimir_desde_grilla (juego->getImagen(IMG_BLOQUES_FIRE),getCuadro()+idTileBloque*6, screen, x,y,4,6,0);
}

// bomba 

#include "bomba.hpp"

Bomba::Bomba(SDL_Surface *imgBomba,int x,int y,int alcance,int lanzador,int id):
    Animacion(imgBomba,1,3,"0,0,0,1,1,1,2,2,2,2,2,2,1,1,1,0,0,0",x,y,id){
    this->lanzador=lanzador;
    this->alcance=alcance;
    type=BOMBA;
    loop=2;
}


// CGRoup 

#include "CGroup.hpp"

Group::Group(Interfaz * parent){
    this->parent=parent;
}


void Group::add(Sprite * nu){
    v_personajes.push_back(nu);
}

/*
void Group::kill(Sprite *del,bool del_mem){
    list<Sprite*>::iterator p_Sprite=find( v_personajes.begin(), v_personajes.end(), del );
    if(p_Sprite!=v_personajes.end()){

        //si se eliminara tambien no solo la referencia del grupo, sino ademas de la memoria
        if(del_mem){
            delete (*p_Sprite);
            (*p_Sprite)=NULL;
        }
        
        v_personajes.erase(p_Sprite);
    }
}*/

/*
bool Group::contain(Sprite * bus){
     //lo vuscamos usando iteradores
    list<Sprite*>::iterator p_Sprite=find( v_personajes.begin(), v_personajes.end(), bus );
    
    //si es distinto del final es porque lo contiene
    return p_Sprite!=v_personajes.end();
}*/
/*
Sprite * Group::collide(Sprite * coli){
    list<Sprite*>::iterator p_Sprite= v_personajes.begin();
    
    while(p_Sprite != v_personajes.end()){
         if(rects_colisionan(coli->rect(),(*p_Sprite)->rect())){
              return (*p_Sprite);
         }
         p_Sprite++;
        
    }
    
    return NULL;
    
}*/
void Group::erase(Sprite * borrar){
     //lo buscamos usando iteradores
    deque<Sprite*>::iterator p_Sprite=find( v_personajes.begin(), v_personajes.end(), borrar );
    if((p_Sprite==v_personajes.end()&&(*p_Sprite)==borrar)||p_Sprite!=v_personajes.end())
        v_personajes.erase(p_Sprite);
}
void Group::update(Uint8 * keys){
    deque<Sprite*>::iterator p_Sprite= v_personajes.begin();
    
    if(!keys)//si no paso las keys las obtenemos
        keys=SDL_GetKeyState(0);
        
    
    while(p_Sprite != v_personajes.end()){
        (*p_Sprite)->update(keys);
         if(p_Sprite != v_personajes.end()){
             if((*p_Sprite)->isMuerto()){
                parent->killSprite((*p_Sprite)->getTipo(),(*p_Sprite)->getId());
                delete(*p_Sprite);
                (*p_Sprite) = 0x0;
                p_Sprite = v_personajes.erase(p_Sprite);
             }else{
                 p_Sprite++;
              }
        }
    }
}

void Group::draw(SDL_Surface * su){

   for (size_t i=0; i<v_personajes.size(); i++)
        v_personajes[i]->draw(su);
}

Group::~Group(){
    v_personajes.clear();
}


//  control animacion 

#include <iostream>
#include "Control_Animacion.h"


using namespace std;


ControlAnimacion::ControlAnimacion(char *frames) {
    if(frames)
        setCuadros(frames);
    else{
    }

}
void ControlAnimacion::setCuadros(char * frames){
    int i = 0;
    char frames_tmp[MAX_NUM_CUADROS];
    char *proximo;    
    
    strncpy(frames_tmp, frames,MAX_NUM_CUADROS);
    // Trabajamos con una copia de los cuadros indicados

    for(proximo = strtok(frames_tmp, ","); proximo; i++){

	// Desmembramos la cadena separada por comas

	this->cuadros[i] = atoi(proximo);
	proximo = strtok(NULL, ",\0");
    }

    // Inicializamos las variables
    
    this->cuadros[i] = -1;
    this->paso = 0;
    inicio=0;
	
#ifdef DEBUG    
    cout << "Control_Animacion::Control_Animacion()" << endl;
#endif
}

int ControlAnimacion::cuadro(void) {
    return cuadros[paso];
}


bool ControlAnimacion::avanzar(void) {	

    if(cuadros[++paso] == -1) {
	   paso = inicio;
	   return true;
    }
    
    return false;
}


void ControlAnimacion::reiniciar(void) {

    // Volvemos al principio

    paso = inicio;
}


bool ControlAnimacion::es_primer_cuadro(void) {

    if(paso == inicio)
	   return true;

    return false;
}

ControlAnimacion::~ControlAnimacion() {

#ifdef DEBUG
    cout << "Control_Animacion::~Control_Animacion()" << endl;
#endif
}


// control player 

#include "control_player.hpp"

using namespace std;

ControlPlayer::ControlPlayer(bool ini){
    if(ini)iniciar();
    
};

void ControlPlayer::iniciar(){
    for(int i=0;i<_TECLAS;i++){
        es_boton_joystick[i]=false;
        es_direccion_joystick[i]=false;
        keys_players[i]=static_cast<SDLKey>(0);
        strcpy(nombres_joysticks[i],"sin asig.");
    }
}

bool ControlPlayer::cargar(char ruta[],bool ini){
    ifstream fs2(ruta,ios::in|ios::binary);
    if(!fs2){
        cerr<<"Error leyendo control en:"<<ruta<<endl;
        if(ini)iniciar();
        return false;
    }else{
        fs2.read(reinterpret_cast<char *> (this),sizeof(ControlPlayer));
        fs2.close();
    }
    return true;
};
void ControlPlayer::guardar(char ruta[]){
    ofstream fs2(ruta,ios::out|ios::binary);    
    fs2.write(reinterpret_cast<char *> (this),sizeof(ControlPlayer));
    fs2.close();
};

char * ControlPlayer::getName(TeclaPlayer tecla){
    return nombres_joysticks[tecla];
};
bool ControlPlayer::isBotonJoystick(TeclaPlayer tecla){
    return es_boton_joystick[tecla];
};
bool ControlPlayer::isDireccionJoystick(int tecla){
    return es_direccion_joystick[tecla];
};
SDLKey ControlPlayer::getKey(TeclaPlayer tecla){
    return keys_players[tecla];
};

void ControlPlayer::setName(TeclaPlayer tecla,const char name[]){
    strcpy(nombres_joysticks[tecla],name);
};
void ControlPlayer::setIsBotonJoystick(TeclaPlayer tecla,bool nuevo){
    es_boton_joystick[tecla]=nuevo;
};
void ControlPlayer::setIsDireccionJoystick(int tecla,bool nuevo){
    es_direccion_joystick[tecla]=nuevo;
};
void ControlPlayer::setKey(TeclaPlayer tecla,SDLKey nuevo){
    keys_players[tecla]=nuevo;
};

void ControlPlayer::setDefaultKeys(IdPlayer id){
    iniciar();
    if(id==PLAYER_1){
    	keys_players[TECLA_IZQUIERDA] = SDLK_a;
    	keys_players[TECLA_DERECHA] = SDLK_d;
    	keys_players[TECLA_ARRIBA] = SDLK_w;
    	keys_players[TECLA_ABAJO] = SDLK_s;
        keys_players[TECLA_ACCION]=SDLK_e;
        keys_players[TECLA_START]=SDLK_q;
//        keys_players[TECLA_ACCION_2]=SDLK_1;
    }else if(id==PLAYER_2){
    	keys_players[TECLA_IZQUIERDA] = SDLK_LEFT;
    	keys_players[TECLA_DERECHA] = SDLK_RIGHT;
    	keys_players[TECLA_ARRIBA] = SDLK_UP;
    	keys_players[TECLA_ABAJO] = SDLK_DOWN;
        keys_players[TECLA_ACCION]=SDLK_KP_ENTER;
        keys_players[TECLA_START]=SDLK_p;
//        keys_players[TECLA_ACCION_2]=SDLK_KP1; 
    }else if(id==PLAYER_3){
    	keys_players[TECLA_IZQUIERDA] = SDLK_f;
    	keys_players[TECLA_DERECHA] = SDLK_h;
    	keys_players[TECLA_ARRIBA] = SDLK_t;
    	keys_players[TECLA_ABAJO] = SDLK_g;
        keys_players[TECLA_ACCION]=SDLK_r;
        keys_players[TECLA_START]=SDLK_y;
//        keys_players[TECLA_ACCION_2]=SDLK_5; 
    
    }else if(id==PLAYER_4){
    	keys_players[TECLA_IZQUIERDA] = SDLK_j;
    	keys_players[TECLA_DERECHA] = SDLK_l;
    	keys_players[TECLA_ARRIBA] = SDLK_i;
    	keys_players[TECLA_ABAJO] = SDLK_k;
        keys_players[TECLA_ACCION]=SDLK_u;
        keys_players[TECLA_START]=SDLK_o;
//        keys_players[TECLA_ACCION_2]=SDLK_8;
        
    }else if(id==PLAYER_5){
    	keys_players[TECLA_IZQUIERDA] = SDLK_1;
    	keys_players[TECLA_DERECHA] = SDLK_2;
    	keys_players[TECLA_ARRIBA] = SDLK_3;
    	keys_players[TECLA_ABAJO] = SDLK_4;
        keys_players[TECLA_ACCION]=SDLK_5;
        keys_players[TECLA_START]=SDLK_6;
//        keys_players[TECLA_ACCION_2]=SDLK_8;

    }
}


// CSprite 

#include "CSprite.hpp"
/*
Sprite::Sprite(Sint16 x,Sint16 y):
        rect(x,y){
}*/
/*
void Sprite::add(Group * nu){
    v_grupos.push_back(nu);
    nu->add(this);
}*/

bool Sprite::isMuerto(){
     return self_kill;
}

void Sprite::kill(){
    self_kill=true;
}
bool Sprite::colision(SDL_Rect & rect_coli){       
    return rects_colisionan(rect,rect_coli);
}


// dat_nivel 

#include "dat_nivel.hpp"

DatNivel::DatNivel(){
    iniciar();
}

void DatNivel::iniciar(){
    for(int i=0;i<_PLAYERS;i++){
       x_init[i]=0;
       y_init[i]=0;
    }
    bombas_ini=vidas_ini=alcance_ini=id_tile=items=0;
}

DatNivel::DatNivel(char ruta[]){
    id_tile=buscar_dato(ruta,"id_tile");
    if(id_tile<0)id_tile=0;
    items=buscar_dato(ruta,"items");
    if(items<0)items=0;
    bombas_ini=buscar_dato(ruta,"bombas");
    if(bombas_ini<0)bombas_ini=0;
    vidas_ini=buscar_dato(ruta,"vidas");
    if(vidas_ini<0)vidas_ini=0;
    alcance_ini=buscar_dato(ruta,"alcance");
    if(alcance_ini<0)alcance_ini=0;
    
    char ruta1[50];
    for(int i=0;i<5;i++){
        sprintf(ruta1,"x_init_player_%d",i+1);
        x_init[i]=buscar_dato(ruta,ruta1);
        if(x_init[i]<0)x_init[i]=0;

        sprintf(ruta1,"y_init_player_%d",i+1);
        y_init[i]=buscar_dato(ruta,ruta1);
        if(y_init[i]<0)y_init[i]=0;
    }
        
}
void DatNivel::guardar(char rutaDestino[]){
    ofstream fnivel(rutaDestino);
    fnivel << "bombas:"<<bombas_ini<<endl;
    fnivel << "vidas:"<<vidas_ini<<endl;
    fnivel << "alcance:"<<alcance_ini<<endl;
    fnivel << "id_tile:"<<id_tile<<endl;
    fnivel << "items:"<<items<<endl;

    for(int i=0;i<5;i++){
        fnivel << "x_init_player_"<<i+1<<":"<<x_init[i]<<endl;
        fnivel << "y_init_player_"<<i+1<<":"<<y_init[i]<<endl;
    }
    fnivel.close();
}


int DatNivel::getX(int id_player){
    return x_init[id_player];
}
int DatNivel::getY(int id_player){
    return y_init[id_player];
}
void DatNivel::setX(int id_player,int nuevo){
    x_init[id_player]=nuevo;
}
void DatNivel::setY(int id_player,int nuevo){
    y_init[id_player]=nuevo;
}
int DatNivel::getBombas(){
    return bombas_ini;
}
int DatNivel::getVidas(){
    return vidas_ini;
}
int DatNivel::getAlcanceBombas(){
    return alcance_ini;
}
int DatNivel::getIdTile(){
    return id_tile;
}
int DatNivel::getNumItems(){
    return items;
}


// editor 

#include "editor.hpp"

Editor::Editor(GameManager * game):
    botonBorrar(game->getImagen(IMG_BOTON_BORRAR_MAPA),this,150,221){
    #ifdef DEBUG
        cout << "Constructor de Editor: "<<this<<endl;
    #endif
    
    this->game=game;
    
    rects_botones[0][EDITOR_BOTON_FLECHA].x=57;
    rects_botones[0][EDITOR_BOTON_FLECHA].y=16;
    rects_botones[0][EDITOR_BOTON_FLECHA].w=21;
    rects_botones[0][EDITOR_BOTON_FLECHA].h=19;
    estados_botones[0][EDITOR_BOTON_FLECHA]=BOTON_NORMAL;

    rects_botones[0][EDITOR_BOTON_GUARDAR].x=235;
    rects_botones[0][EDITOR_BOTON_GUARDAR].y=221;
    rects_botones[0][EDITOR_BOTON_GUARDAR].w=81;
    rects_botones[0][EDITOR_BOTON_GUARDAR].h=18;
    estados_botones[0][EDITOR_BOTON_GUARDAR]=BOTON_NORMAL;

    botonBorrar.bindAccion(&Editor::borrarMapa);
//    botonBorrar.bindAccion(borrarMapaNoEstatico);
    rects_botones[EDITOR_MODIF_BOMBA][EDITOR_BOTON_MAS].x=88;
    rects_botones[EDITOR_MODIF_BOMBA][EDITOR_BOTON_MAS].y=5;
    rects_botones[EDITOR_MODIF_BOMBA][EDITOR_BOTON_MAS].w=17;
    rects_botones[EDITOR_MODIF_BOMBA][EDITOR_BOTON_MAS].h=10;
    estados_botones[EDITOR_MODIF_BOMBA][EDITOR_BOTON_MAS]=BOTON_NORMAL;

    rects_botones[EDITOR_MODIF_BOMBA][EDITOR_BOTON_MENOS].x=111;
    rects_botones[EDITOR_MODIF_BOMBA][EDITOR_BOTON_MENOS].y=5;
    rects_botones[EDITOR_MODIF_BOMBA][EDITOR_BOTON_MENOS].w=17;
    rects_botones[EDITOR_MODIF_BOMBA][EDITOR_BOTON_MENOS].h=10;
    estados_botones[EDITOR_MODIF_BOMBA][EDITOR_BOTON_MENOS]=BOTON_NORMAL;


    rects_botones[EDITOR_MODIF_EXPLOSION][EDITOR_BOTON_MAS].x=151;
    rects_botones[EDITOR_MODIF_EXPLOSION][EDITOR_BOTON_MAS].y=5;
    rects_botones[EDITOR_MODIF_EXPLOSION][EDITOR_BOTON_MAS].w=17;
    rects_botones[EDITOR_MODIF_EXPLOSION][EDITOR_BOTON_MAS].h=10;
    estados_botones[EDITOR_MODIF_EXPLOSION][EDITOR_BOTON_MAS]=BOTON_NORMAL;

    rects_botones[EDITOR_MODIF_EXPLOSION][EDITOR_BOTON_MENOS].x=173;
    rects_botones[EDITOR_MODIF_EXPLOSION][EDITOR_BOTON_MENOS].y=5;
    rects_botones[EDITOR_MODIF_EXPLOSION][EDITOR_BOTON_MENOS].w=17;
    rects_botones[EDITOR_MODIF_EXPLOSION][EDITOR_BOTON_MENOS].h=10;
    estados_botones[EDITOR_MODIF_EXPLOSION][EDITOR_BOTON_MENOS]=BOTON_NORMAL;


    rects_botones[EDITOR_MODIF_VIDAS][EDITOR_BOTON_MAS].x=215;
    rects_botones[EDITOR_MODIF_VIDAS][EDITOR_BOTON_MAS].y=5;
    rects_botones[EDITOR_MODIF_VIDAS][EDITOR_BOTON_MAS].w=17;
    rects_botones[EDITOR_MODIF_VIDAS][EDITOR_BOTON_MAS].h=10;
    estados_botones[EDITOR_MODIF_VIDAS][EDITOR_BOTON_MAS]=BOTON_NORMAL;

    rects_botones[EDITOR_MODIF_VIDAS][EDITOR_BOTON_MENOS].x=237;
    rects_botones[EDITOR_MODIF_VIDAS][EDITOR_BOTON_MENOS].y=5;
    rects_botones[EDITOR_MODIF_VIDAS][EDITOR_BOTON_MENOS].w=17;
    rects_botones[EDITOR_MODIF_VIDAS][EDITOR_BOTON_MENOS].h=10;
    estados_botones[EDITOR_MODIF_VIDAS][EDITOR_BOTON_MENOS]=BOTON_NORMAL;


    rects_botones[EDITOR_MODIF_ITEMS][EDITOR_BOTON_MAS].x=269;
    rects_botones[EDITOR_MODIF_ITEMS][EDITOR_BOTON_MAS].y=5;
    rects_botones[EDITOR_MODIF_ITEMS][EDITOR_BOTON_MAS].w=17;
    rects_botones[EDITOR_MODIF_ITEMS][EDITOR_BOTON_MAS].h=10;
    estados_botones[EDITOR_MODIF_ITEMS][EDITOR_BOTON_MAS]=BOTON_NORMAL;

    rects_botones[EDITOR_MODIF_ITEMS][EDITOR_BOTON_MENOS].x=292;
    rects_botones[EDITOR_MODIF_ITEMS][EDITOR_BOTON_MENOS].y=5;
    rects_botones[EDITOR_MODIF_ITEMS][EDITOR_BOTON_MENOS].w=17;
    rects_botones[EDITOR_MODIF_ITEMS][EDITOR_BOTON_MENOS].h=10;
    estados_botones[EDITOR_MODIF_ITEMS][EDITOR_BOTON_MENOS]=BOTON_NORMAL;

    //para la ventana 1
    rects_botones_elegir_terreno[EDITOR_FLECHA_IZQUIERDA].x=17;
    rects_botones_elegir_terreno[EDITOR_FLECHA_IZQUIERDA].y=67;
    rects_botones_elegir_terreno[EDITOR_FLECHA_IZQUIERDA].w=60;
    rects_botones_elegir_terreno[EDITOR_FLECHA_IZQUIERDA].h=27;
    estados_botones_elegir_terreno[EDITOR_FLECHA_IZQUIERDA]=BOTON_NORMAL;

    rects_botones_elegir_terreno[EDITOR_FLECHA_DERECHA].x=239;
    rects_botones_elegir_terreno[EDITOR_FLECHA_DERECHA].y=67;
    rects_botones_elegir_terreno[EDITOR_FLECHA_DERECHA].w=60;
    rects_botones_elegir_terreno[EDITOR_FLECHA_DERECHA].h=27;
    estados_botones_elegir_terreno[EDITOR_FLECHA_DERECHA]=BOTON_NORMAL;

    rects_botones_elegir_terreno[EDITOR_MAPA_1].x=26;
    rects_botones_elegir_terreno[EDITOR_MAPA_1].y=102;
    rects_botones_elegir_terreno[EDITOR_MAPA_1].w=111;
    rects_botones_elegir_terreno[EDITOR_MAPA_1].h=121;
    estados_botones_elegir_terreno[EDITOR_MAPA_1]=BOTON_NORMAL;

    rects_botones_elegir_terreno[EDITOR_MAPA_2].x=177;
    rects_botones_elegir_terreno[EDITOR_MAPA_2].y=102;
    rects_botones_elegir_terreno[EDITOR_MAPA_2].w=111;
    rects_botones_elegir_terreno[EDITOR_MAPA_2].h=121;
    estados_botones_elegir_terreno[EDITOR_MAPA_2]=BOTON_NORMAL;
    boton_visible[EDITOR_MAPA_1]=1;

    SDL_ShowCursor(SDL_ENABLE);

    tile_activo=-1;
    id_nivel=0;


    maxTerrenoBatalla=buscar_dato(RUTA_CONFIG_BASE,"MaxTerreno");
    //realizamos las previews de los niveles ya creados
    player_activo=PLAYER_NONE;
    previews_niveles=0;
    crearReferencias();
    game->playSonido(SND_EDITOR);

    cambiarVentana(EDITOR_ABRIR_NIVEL);

}

void Editor::crearReferencias(){
    char ruta[50];
    if(previews_niveles){
         for(int i=0;i<maxTerrenoBatalla;i++){
            #ifdef DEBUG
                cout << "liberando Surface:"<<previews_niveles[i]<<endl;
            #endif
             SDL_FreeSurface(previews_niveles[i]);
            previews_niveles[i]=NULL;
         }
        #ifdef DEBUG
            cout << "liberando Surface:"<<previews_niveles[maxTerrenoBatalla]<<endl;
        #endif
         if(botonBorrar.getVisible())SDL_FreeSurface(previews_niveles[maxTerrenoBatalla]);
         previews_niveles[maxTerrenoBatalla]=NULL;
        #ifdef DEBUG
            cout << "liberando Previews:"<<previews_niveles<<endl;
        #endif
        delete [] previews_niveles;
        previews_niveles=NULL;
    }
    previews_niveles=new SDL_Surface*[maxTerrenoBatalla + 1];
    #ifdef DEBUG
        cout << "creada Previews:"<<previews_niveles<<endl;
    #endif

    DatNivel * data2;
    SDL_Surface * img_players[5]={game->getImagen(IMG_PLAYER_1),
                                  game->getImagen(IMG_PLAYER_2),
                                  game->getImagen(IMG_PLAYER_3),
                                  game->getImagen(IMG_PLAYER_4),
                                  game->getImagen(IMG_PLAYER_5)};
    for(int i=0;i<maxTerrenoBatalla;i++){
        sprintf(ruta,"data/niveles/batalla/%d.txt",i+ 1);
        data2=new DatNivel(ruta);
        sprintf(ruta,"data/niveles/batalla/%d.map",i+ 1);
        previews_niveles[i]=Mapa::getPreviewTerreno(ruta,data2,game->getImagen(IMG_TILES),img_players,8,40);
    #ifdef DEBUG
        cout << "Creada Surface: "<<previews_niveles[i]<<endl;
    #endif
        delete data2;
    }
    //realizamos la preview del "nuevo nivel"

    sprintf(ruta,"data/niveles/batalla/%d.map",NIVEL_BASE);
    previews_niveles[maxTerrenoBatalla]=Mapa::getPreviewTerreno(ruta,NULL,game->getImagen(IMG_TILES),NULL,8,40);
    #ifdef DEBUG
        cout << "Creada Surface: "<<previews_niveles[maxTerrenoBatalla]<<endl;
    #endif

}
void Editor::cambiarVentana(int nueva){
    ventana=nueva;

    if(ventana==EDITOR_ABRIR_NIVEL)cambiarPagina(1);
}

void Editor::cambiarPagina(unsigned int num_pagina){
        pagina=num_pagina;

        if(pagina==1){
            boton_visible[EDITOR_FLECHA_IZQUIERDA]=false;
        }else{
            boton_visible[EDITOR_FLECHA_IZQUIERDA]=true;
            }

        if((maxTerrenoBatalla+1)-pagina*2<=0){
            boton_visible[EDITOR_FLECHA_DERECHA]=false;
        }else{
            boton_visible[EDITOR_FLECHA_DERECHA]=true;
            }

        if((maxTerrenoBatalla+1)-pagina*2==-1){
            boton_visible[EDITOR_MAPA_2]=false;
        }else{
            boton_visible[EDITOR_MAPA_2]=true;
            }

}

void Editor::iniciarEdicion(int id){
    tile_activo=-1;
    player_activo=PLAYER_NONE;
    mantiene_presionado=false;
    id_nivel=id;
    char ruta[40];
    
    if(id !=NIVEL_BASE){
        sprintf(ruta,"data/niveles/batalla/%d.map",id + 1);
        Mapa::cargarMapaDeArchivoBin(ruta,mapa);
        sprintf(ruta,"data/niveles/batalla/%d.txt",id + 1);
        
        if(data)delete data;
        data=new DatNivel(ruta);
        botonBorrar.setVisible(true);
        leerInfTile();
        
    }else{
        botonBorrar.setVisible(false);
        sprintf(ruta,"data/niveles/batalla/%d.map",NIVEL_BASE);
        Mapa::cargarMapaDeArchivoBin(ruta,mapa);
        if(data)delete data;
        data=new DatNivel();
        sprintf(ruta,"data/niveles/batalla/%d.map",NIVEL_BASE + 1);  
        data->setBombas(1);
        data->setAlcanceBombas(1);
         
        data->setX(PLAYER_1,X_INIT_PLAYER_1);
        data->setY(PLAYER_1,Y_INIT_PLAYER_1);

        data->setX(PLAYER_2,X_INIT_PLAYER_2);
        data->setY(PLAYER_2,Y_INIT_PLAYER_2);

        data->setX(PLAYER_3,X_INIT_PLAYER_3);
        data->setY(PLAYER_3,Y_INIT_PLAYER_3);

        data->setX(PLAYER_4,X_INIT_PLAYER_4);
        data->setY(PLAYER_4,Y_INIT_PLAYER_4);

        data->setX(PLAYER_5,X_INIT_PLAYER_5);
        data->setY(PLAYER_5,Y_INIT_PLAYER_5);
        EjeX=25;
        EjeY=58;
        yTablero=8;
        idFondo=IMG_FONDO_METAL;
        
     }

    cambiarVentana(EDICION_NIVEL);
}
void Editor::leerInfTile(){
    char key[50],valor[50];
    sprintf(key,"data/imagenes/objetos/tile_%d.txt",data->getIdTile() + 1);
    ifstream ftile(key);
    while(!ftile.eof()){
        ftile >>key;
        if(!strcmp(key,"IMG_FONDO")){
            ftile>>valor;
            if(!strcmp(valor,"FONDO_1"))
                idFondo=IMG_FONDO_PARTI;
            else if(!strcmp(valor,"FONDO_2"))
                idFondo=IMG_FONDO_EDIFICIOS;
            else if(!strcmp(valor,"FONDO_3"))
                idFondo=IMG_FONDO_METAL;
        }else if(!strcmp(key,"EJES")){
            ftile>>EjeX>>EjeY;
        }else if(!strcmp(key,"Y_TABLERO")){
            ftile>>yTablero;
        }else{
            ftile>>valor;
        }
    }
    ftile.close();
    
}
void Editor::guardarMapa(){
    char ruta[50];
    int id_file=id_nivel+1;
    if(id_nivel ==NIVEL_BASE){
        id_file=++maxTerrenoBatalla;
    }
        
    sprintf(ruta,"data/niveles/batalla/%d.map",id_file);
    ofstream fs2(ruta,ios::out|ios::binary);
    if(!fs2){cout << "Error guardando mapa en archivo:"<<ruta<<endl;return;}
    
    fs2.write(reinterpret_cast<char *> (mapa),Mapa::MAXMAP);
    fs2.close();

    sprintf(ruta,"data/niveles/batalla/%d.txt",id_file);
    data->guardar(ruta);
    
    int puntaje=buscar_dato(RUTA_CONFIG_BASE,"Puntaje");;
    ofstream file(RUTA_CONFIG_BASE);
    file << "MaxTerreno:"<<maxTerrenoBatalla<<endl;
    file << "Puntaje:"<<puntaje<<endl;
    file.close();
    crearReferencias();
}

void Editor::borrarMapa(){
    char ruta[50],nuevaRuta[50];
    int id_file=id_nivel+1;
    
    sprintf(ruta,"data/niveles/batalla/%d.map",id_file);
    //cout << "Borrar:"<<ruta<<endl;
    if(remove(ruta)){
        cout <<"Error borrando: "<<ruta<<endl;
    }
    
    sprintf(ruta,"data/niveles/batalla/%d.txt",id_file);
    //cout << "Borrar:"<<ruta<<endl;
    if(remove(ruta)){
        cout <<"Error borrando: "<<ruta<<endl;
        return;
    }
    /*Función auxiliar no estatica para poder usar el boton*/
    maxTerrenoBatalla--;
    int puntaje=buscar_dato(RUTA_CONFIG_BASE,"Puntaje");;
    ofstream file(RUTA_CONFIG_BASE);
    file << "MaxTerreno:"<<maxTerrenoBatalla<<endl;
    file << "Puntaje:"<<puntaje<<endl;
    file.close();
    
    for(int i=0;i<maxTerrenoBatalla+1-id_file;i++){
        sprintf(ruta,"data/niveles/batalla/%d.map",id_file+i+1);
        sprintf(nuevaRuta,"data/niveles/batalla/%d.map",id_file+i);
        //cout << "Renombrando:"<<ruta<<" a :"<<nuevaRuta<<endl;
	    rename(ruta,nuevaRuta);

        sprintf(ruta,"data/niveles/batalla/%d.txt",id_file+i+1);
        sprintf(nuevaRuta,"data/niveles/batalla/%d.txt",id_file+i);
        //cout << "Renombrando:"<<ruta<<" a :"<<nuevaRuta<<endl;
	    rename(ruta,nuevaRuta);

    }
    
    crearReferencias();
    cambiarVentana(EDITOR_ABRIR_NIVEL);

}



bool Editor::EditPointMap(SDL_Event * evento){
    
    if(punto_en_rect_coordenadas(evento->motion.x,evento->motion.y,EjeX,EjeY,272,176)&&tile_activo!=-1){
        int x=(evento->motion.x- EjeX)/16 ,y=(evento->motion.y - EjeY)/16 ; //calculamos la posicion del cursor respecto a los tiles
        if(mapa[y*Mapa::COLUMNAS+x]!=tile_activo){//si no es el mismo
            /*reproducimos un sonido de acuerdo al bloque anterior que estaba ahy*/
            if(mapa[y*Mapa::COLUMNAS+x]!=Mapa::BLOQUE_PISO&&mapa[y*Mapa::COLUMNAS+x]!=Mapa::BLOQUE_PISO_SOMBRA){//si ya estaba ocupado
                game->play(SFX_TONO_SECO);
            }
            game->play(SFX_TONO_ACUATICO);
            if((tile_activo==Mapa::BLOQUE_PISO&&evento->motion.y-16>=EjeY&&mapa[((evento->motion.y -16- EjeY)/16)*Mapa::COLUMNAS+x]!=Mapa::BLOQUE_PISO&&mapa[((evento->motion.y -16- EjeY)/16)*Mapa::COLUMNAS+x]!=Mapa::BLOQUE_PISO_SOMBRA)||(y==0&&tile_activo==Mapa::BLOQUE_PISO)||(tile_activo==Mapa::BLOQUE_PISO&&mapa[y*Mapa::COLUMNAS+x]==Mapa::BLOQUE_PISO_SOMBRA))
                mapa[y*Mapa::COLUMNAS+x]=Mapa::BLOQUE_PISO_SOMBRA;
            else
                mapa[y*Mapa::COLUMNAS+x]=tile_activo;//MODIFICAMOS EL MAPA
            
            if(tile_activo!=Mapa::BLOQUE_PISO){
                if(evento->motion.y+16!=EjeY+Mapa::FILAS*16&&mapa[((evento->motion.y +16- EjeY)/16)*Mapa::COLUMNAS+x]==Mapa::BLOQUE_PISO){
                    mapa[((evento->motion.y +16- EjeY)/16)*Mapa::COLUMNAS+x]=Mapa::BLOQUE_PISO_SOMBRA;
                }
            }else{
                if(evento->motion.y+16<=EjeY+Mapa::FILAS*16&&mapa[((evento->motion.y +16- EjeY)/16)*Mapa::COLUMNAS+x]==Mapa::BLOQUE_PISO_SOMBRA){
                    mapa[((evento->motion.y +16- EjeY)/16)*Mapa::COLUMNAS+x]=Mapa::BLOQUE_PISO;
                }
            }
            return true;
        }else
            return true;//REGRESAMOS "TRUE" PORQUE AL MENOS PASO LA PRIMERA PRUEBA SOLO QUE EN ESE PUNTO YA ESTABA EL MISMO TILE
    }
    return false;
}
void Editor::procesarEvento(SDL_Event * evento){
        static int i,j;

        switch(ventana){

            case EDICION_NIVEL:
                botonBorrar.procesarEvento(evento);
                if(evento->type==SDL_KEYDOWN&&evento->key.keysym.sym==SDLK_ESCAPE){
                    cambiarVentana(EDITOR_ABRIR_NIVEL);
                }else if(evento->type == SDL_MOUSEMOTION){
                    for(i=0;i<5;i++){
                        for(j=0;j<2;j++){
                            if(i==0&&j==EDITOR_BOTON_GUARDAR)continue;
                        	if(punto_en_rect(evento->motion.x,evento->motion.y-yTablero,&rects_botones[i][j])){
                                if(estados_botones[i][j]!=BOTON_PRESIONADO)
                        				estados_botones[i][j]=BOTON_RESALTADO;
                            }
                			else
                				estados_botones[i][j]=BOTON_NORMAL;
                        }
                    }
                    
                	if(punto_en_rect(evento->motion.x,evento->motion.y,&rects_botones[0][EDITOR_BOTON_GUARDAR])){
                        if(estados_botones[0][EDITOR_BOTON_GUARDAR]!=BOTON_PRESIONADO)
                				estados_botones[0][EDITOR_BOTON_GUARDAR]=BOTON_RESALTADO;
                    }
        			else
        				estados_botones[0][EDITOR_BOTON_GUARDAR]=BOTON_NORMAL;
                    
                    if(player_activo>=0){

                        if(evento->motion.y+10<EjeY||evento->motion.y+10+10>EjeY+Mapa::FILAS*16||evento->motion.x+3<EjeX||evento->motion.x+3 + 10>EjeX+Mapa::COLUMNAS*16){
                            int x=evento->motion.x,y=evento->motion.y;

                            if(evento->motion.y+10<EjeY){
                                y=EjeY-10;
                            }else if(evento->motion.y+21>EjeY+Mapa::FILAS*16){
                                y=EjeY+Mapa::FILAS*16-21;
                            }

                            if(evento->motion.x+3<EjeX){
                                x=EjeX-3;
                            }else if(evento->motion.x+13>EjeX+Mapa::COLUMNAS*16){
                                x=EjeX+Mapa::COLUMNAS*16-13;
                            }

                            SDL_WarpMouse(x,y);
                        }


                    }

                    if(mantiene_presionado&&tile_activo!=-1)
                        EditPointMap(evento);
                            
                }else if(evento->type == SDL_MOUSEBUTTONDOWN) {
                    if(evento->button.button==SDL_BUTTON_LEFT){
                            for(i=0;i<5;i++)
                                for(j=0;j<2;j++){
                                    if(i==0&&j==EDITOR_BOTON_GUARDAR)continue;
                                	if(punto_en_rect(evento->motion.x,evento->motion.y-yTablero,&rects_botones[i][j])){
                            			estados_botones[i][j]=BOTON_PRESIONADO;
                                        game->play(SFX_ESTRANYO);
                                    }
                                }
                                
                        	if(punto_en_rect(evento->motion.x,evento->motion.y,&rects_botones[0][EDITOR_BOTON_GUARDAR])){
                    			estados_botones[0][EDITOR_BOTON_GUARDAR]=BOTON_PRESIONADO;
                                game->play(SFX_ESTRANYO);
                            }



                            if(punto_en_rect_coordenadas(evento->motion.x,evento->motion.y,6,20+yTablero,47,16)){
                                tile_activo=(evento->motion.x-6)/16;
                                if(tile_activo==2)tile_activo=3;
                                player_activo=PLAYER_NONE;
                                game->play(SFX_CAMPANADA);
                            }
                            else
                                if(EditPointMap(evento))
                                    mantiene_presionado=1;


                        }else if(evento->button.button==SDL_BUTTON_RIGHT) {
                            for(i=0;i<_PLAYERS;i++)
                                if(punto_en_rect_coordenadas(evento->motion.x,evento->motion.y,data->getX((IdPlayer)i),data->getY((IdPlayer)i),15,15)){
                                    player_activo=(IdPlayer)i;
                                    tile_activo=-1;
                                    return;
                                }

                            if(player_activo!=PLAYER_NONE){
                                data->setX(player_activo,evento->motion.x);
                                data->setY(player_activo,evento->motion.y);
                                player_activo=PLAYER_NONE;
                            }
                        }


            	}
            	else if(evento->type == SDL_MOUSEBUTTONUP&&evento->button.button==SDL_BUTTON_LEFT) {
                    mantiene_presionado=false;
                        for(i=0;i<5;i++)
                            for(j=0;j<2;j++){
                            	if(punto_en_rect(evento->motion.x,evento->motion.y-yTablero,&rects_botones[i][j])){
                                        switch(i){
                                            case 0:
                                                if(j==EDITOR_BOTON_FLECHA){
                                                    if(data->getIdTile() < 5){
                                                        data->setIdTile(data->getIdTile()+1);
                                                        if(data->getIdTile()==4){
                                                            data->setIdTile(0);
                                                        }
                                                        leerInfTile();
                                                    }
                                                }
                                                break;
                                            case EDITOR_MODIF_BOMBA:
                                                if(j==EDITOR_BOTON_MAS){
                                                    data->setBombas(data->getBombas()+1);
                                                    if(data->getBombas()>MAX_BOMBAS)
                                                        data->setBombas(MAX_BOMBAS);
                                                }else{
                                                    data->setBombas(data->getBombas()-1);
                                                    if(data->getBombas()<0)
                                                        data->setBombas(0);
                                                }

                                                break;
                                            case EDITOR_MODIF_EXPLOSION:
                                                if(j==EDITOR_BOTON_MAS){
                                                    data->setAlcanceBombas(data->getAlcanceBombas()+1);
                                                    if(data->getAlcanceBombas()>12)
                                                        data->setAlcanceBombas(12);
                                                }else{
                                                    data->setAlcanceBombas(data->getAlcanceBombas()-1);
                                                    if(data->getAlcanceBombas()<0)
                                                        data->setAlcanceBombas(0);
                                                }   
                                                break;
                                            case EDITOR_MODIF_VIDAS:
                                                if(j==EDITOR_BOTON_MAS){
                                                    data->setVidas(data->getVidas()+1);
                                                    if(data->getVidas()>5)
                                                        data->setVidas(5);
                                                }else{
                                                    data->setVidas(data->getVidas()-1);
                                                    if(data->getVidas()<0)
                                                        data->setVidas(0);
                                                }
                                                break;
                                            case EDITOR_MODIF_ITEMS:
                                                if(j==EDITOR_BOTON_MAS){
                                                    data->setNumItems(data->getNumItems()+1);
                                                    if(data->getNumItems()>30)
                                                        data->setNumItems(30);
                                                }else{
                                                    data->setNumItems(data->getNumItems()-1);
                                                    if(data->getNumItems()<0)
                                                        data->setNumItems(0);
                                                }
                                                break;
                                        }
                            			estados_botones[i][j]=BOTON_NORMAL;
                                }//fin si punto rect
                        }//seg for
                    if(punto_en_rect(evento->motion.x,evento->motion.y,&rects_botones[0][EDITOR_BOTON_GUARDAR])){
                        guardarMapa();
                        cambiarVentana(EDITOR_ABRIR_NIVEL);
                    }
                }//fin else if
                
            break;

        case EDITOR_ABRIR_NIVEL:
                if(evento->type==SDL_KEYDOWN){
                    switch(evento->key.keysym.sym){
                            case SDLK_ESCAPE:
                                game->cambiarInterfaz(new Menu(game));
                                break;
                            case SDLK_1:
                                 if(boton_visible[EDITOR_MAPA_1]){
                                    if((maxTerrenoBatalla+1)-pagina*2!=-1)
                                        iniciarEdicion((pagina-1)*2);
                                    else
                                        iniciarEdicion(NIVEL_BASE);
                                 }
                                break;
                            case SDLK_2:
                                 if(boton_visible[EDITOR_MAPA_2]){
                                    if((maxTerrenoBatalla+1)-pagina*2>0)
                                        iniciarEdicion((pagina-1)*2+1);
                                    else
                                        iniciarEdicion(NIVEL_BASE);
                                    }
                                break;
                            case SDLK_LEFT:
                                 if(boton_visible[EDITOR_FLECHA_IZQUIERDA])
                                       cambiarPagina(pagina - 1);
                                break;
                            case SDLK_RIGHT:
                                 if(boton_visible[EDITOR_FLECHA_DERECHA])
                                       cambiarPagina(pagina + 1);
                                break;
                            default:
                                break;
                    }                                       

                }else if(evento->type == SDL_MOUSEMOTION){
                            if(boton_visible[EDITOR_FLECHA_IZQUIERDA])
                            {
                                if(punto_en_rect(evento->motion.x,evento->motion.y,&rects_botones_elegir_terreno[EDITOR_FLECHA_IZQUIERDA]))
                                    estados_botones_elegir_terreno[EDITOR_FLECHA_IZQUIERDA]=BOTON_RESALTADO;
                                else
                                    estados_botones_elegir_terreno[EDITOR_FLECHA_IZQUIERDA]=BOTON_NORMAL;
                            }

                            if(boton_visible[EDITOR_FLECHA_DERECHA])
                            {
                                if(punto_en_rect(evento->motion.x,evento->motion.y,&rects_botones_elegir_terreno[EDITOR_FLECHA_DERECHA]))
                                    estados_botones_elegir_terreno[EDITOR_FLECHA_DERECHA]=BOTON_RESALTADO;
                                else
                                    estados_botones_elegir_terreno[EDITOR_FLECHA_DERECHA]=BOTON_NORMAL;
                            }



                }else if(evento->type == SDL_MOUSEBUTTONDOWN&&evento->button.button==SDL_BUTTON_LEFT) {
                        for(i=0;i<4;i++)
                            	if(boton_visible[i] && punto_en_rect(evento->motion.x,evento->motion.y,&rects_botones_elegir_terreno[i])){
                            			estados_botones_elegir_terreno[i]=BOTON_PRESIONADO;
                                        game->play(SFX_TONO_SECO);
                                }
            	}
            	else if(evento->type == SDL_MOUSEBUTTONUP&&evento->button.button==SDL_BUTTON_LEFT) {
                        for(i=0;i<4;i++){
                                if(boton_visible[i] && punto_en_rect(evento->motion.x,evento->motion.y,&rects_botones_elegir_terreno[i])){
                                    switch(i){

                                        case EDITOR_FLECHA_IZQUIERDA:
                                            cambiarPagina(pagina - 1);
                                            break;
                                        case EDITOR_FLECHA_DERECHA:
                                            cambiarPagina(pagina + 1);
                                            break;
                                        case EDITOR_MAPA_1:
                                                if((maxTerrenoBatalla+1)-pagina*2!=-1)
                                                    iniciarEdicion((pagina-1)*2);
                                                else
                                                    iniciarEdicion(NIVEL_BASE);
                                            break;
                                        case EDITOR_MAPA_2:
                                                if((maxTerrenoBatalla+1)-pagina*2>0)
                                                    iniciarEdicion((pagina-1)*2+1);
                                                else
                                                    iniciarEdicion(NIVEL_BASE);

                                            break;

                                    }
                                    estados_botones_elegir_terreno[i]=BOTON_NORMAL;
                                }
                            }
                }
            break;
    }

}

void Editor::draw(SDL_Surface * screen){
    static int i,j;
    static char stock[30];

    switch(ventana){
        case EDICION_NIVEL:
                dibujar_objeto(game->getImagen((CodeImagen)(idFondo)),0,0,screen);
                Mapa::draw(screen,game->getImagen(IMG_TILES),mapa,EjeX,EjeY,data->getIdTile());

                dibujar_objeto(game->getImagen(IMG_TABLERO),0,yTablero,screen);

                for(i=1;i<5;i++){
                    for(j=0;j<2;j++){
                        imprimir_desde_grilla(game->getImagen((CodeImagen)(IMG_BOTON_MAS +j)),estados_botones[i][j],screen,rects_botones[i][j].x,rects_botones[i][j].y+yTablero,3,1,0);
                        }
                    }

            
                dibujar_objeto(game->getImagen(IMG_LLAMA),143,20+yTablero,screen);//explosion
                dibujar_objeto(game->getImagen(IMG_BOMBA_PEQUE),91,20+yTablero,screen);//bomba
                dibujar_objeto(game->getImagen(IMG_CORAZON),217,20+yTablero,screen);//vidas
                dibujar_objeto(game->getImagen(IMG_PREGUNTA),270,21+yTablero,screen);//item

                dibujar_objeto(game->getImagen(IMG_TXT_TILES),6,3+yTablero,screen);

                //dibujamos la cantidad de bombas iniciales
                sprintf(stock,"x%d",data->getBombas());
                imprimir_palabra(screen,game->getImagen(IMG_FUENTE_2),107,21+yTablero,stock,STR_NORMAL);

                //dibujamos el alcance de las bombas iniciales
                sprintf(stock,"x%d",data->getAlcanceBombas());
                imprimir_palabra(screen,game->getImagen(IMG_FUENTE_2),177,21+yTablero,stock,STR_NORMAL);

                //dibujamos las vidas iniciales
                sprintf(stock,"x%d",data->getVidas());
                imprimir_palabra(screen,game->getImagen(IMG_FUENTE_2),237,21+yTablero,stock,STR_NORMAL);

                //dibujamos la cantidad de items iniciales
                sprintf(stock,"x%d",data->getNumItems());
                imprimir_palabra(screen,game->getImagen(IMG_FUENTE_2),288,21+yTablero,stock,STR_NORMAL);
                
                for(int j=0;j<3;j++)
                    imprimir_desde_grilla(game->getImagen(IMG_TILES),data->getIdTile()*4+((j!=2)?j:Mapa::BLOQUE_PISO),screen,j*16+6,20+yTablero,4,4,0);

                imprimir_desde_grilla(game->getImagen(IMG_BOTON_FLECHA_PEQUE_DERECHA),estados_botones[0][0],screen,rects_botones[0][0].x,rects_botones[0][0].y+yTablero,3,1,0);

                static int x,y;
                SDL_GetMouseState(&x,&y);

                for(i=0;i<_PLAYERS;i++){
                    if(i!=player_activo){
                        imprimir_desde_grilla(game->getImagen((CodeImagen)(IMG_PLAYER_1 +i)),6,screen,data->getX((IdPlayer)i),data->getY((IdPlayer)i),1,12,1);
                    }else{
                        imprimir_desde_grilla(game->getImagen((CodeImagen)(IMG_PLAYER_1 +i)),6,screen,x,y,1,12,1);
                    }
                }

                if(tile_activo>=0&&punto_en_rect_coordenadas(x,y,EjeX,EjeY,272,176)){
                    static SDL_Rect rect;
                    rect.x=(x-EjeX)/16*16 + EjeX;
                    rect.y=(y-EjeY)/16*16 + EjeY;
                    imprimir_desde_grilla(game->getImagen(IMG_TILES),data->getIdTile()*4+tile_activo,screen,rect.x,rect.y,4,4,1);

                }

                imprimir_desde_grilla(game->getImagen(IMG_BOTON_GUARDAR),estados_botones[0][1],screen,rects_botones[0][1].x,rects_botones[0][1].y,3,1,0);
                botonBorrar.draw(screen);
            break;
        case EDITOR_ABRIR_NIVEL:

                dibujar_objeto(game->getImagen(IMG_FONDO_EDITOR_SELECT_FILE),0,0,screen);


                if(boton_visible[EDITOR_FLECHA_IZQUIERDA])
                    imprimir_desde_grilla(game->getImagen(IMG_BOTON_FLECHA_GRANDE_IZQUIERDA),estados_botones_elegir_terreno[EDITOR_FLECHA_IZQUIERDA],screen,rects_botones_elegir_terreno[EDITOR_FLECHA_IZQUIERDA].x,rects_botones_elegir_terreno[EDITOR_FLECHA_IZQUIERDA].y,3,1,0);

                if(boton_visible[EDITOR_FLECHA_DERECHA])
                    imprimir_desde_grilla(game->getImagen(IMG_BOTON_FLECHA_GRANDE_DERECHA),estados_botones_elegir_terreno[EDITOR_FLECHA_DERECHA],screen,rects_botones_elegir_terreno[EDITOR_FLECHA_DERECHA].x,rects_botones_elegir_terreno[EDITOR_FLECHA_DERECHA].y,3,1,0);


                imprimir_desde_grilla(game->getImagen(IMG_BOTON_ESTRANIO),(estados_botones_elegir_terreno[EDITOR_MAPA_1]==BOTON_PRESIONADO)?1:0,screen,rects_botones_elegir_terreno[EDITOR_MAPA_1].x,rects_botones_elegir_terreno[EDITOR_MAPA_1].y,2,1,0);
                dibujar_objeto(previews_niveles[(pagina-1)*2],rects_botones_elegir_terreno[EDITOR_MAPA_1].x+7,rects_botones_elegir_terreno[EDITOR_MAPA_1].y+5,screen);


                if((maxTerrenoBatalla+1)-pagina*2==-1){
                    imprimir_palabra(screen,game->getImagen(IMG_FUENTE_5),
                                    rects_botones_elegir_terreno[EDITOR_MAPA_1].x+2,
                                    rects_botones_elegir_terreno[EDITOR_MAPA_1].y+50,
                                    "nuevo",STR_NORMAL);

                }
                if(boton_visible[EDITOR_MAPA_2]){
                        imprimir_desde_grilla(game->getImagen(IMG_BOTON_ESTRANIO),(estados_botones_elegir_terreno[EDITOR_MAPA_2]==BOTON_PRESIONADO)?1:0,screen,rects_botones_elegir_terreno[EDITOR_MAPA_2].x,rects_botones_elegir_terreno[EDITOR_MAPA_2].y,2,1,0);
                        dibujar_objeto(previews_niveles[(pagina-1)*2 + 1],rects_botones_elegir_terreno[EDITOR_MAPA_2].x+7,rects_botones_elegir_terreno[EDITOR_MAPA_2].y+5,screen);
    
                        if((maxTerrenoBatalla+1)-pagina*2<=0)
                            imprimir_palabra(screen,game->getImagen(IMG_FUENTE_5),
                                            rects_botones_elegir_terreno[EDITOR_MAPA_2].x+2,
                                            rects_botones_elegir_terreno[EDITOR_MAPA_2].y+50,
                                            "nuevo",STR_NORMAL);
                }
            break;
    }
}

Editor::~Editor(){
    #ifdef DEBUG
        cout << "Llamando al destructor de Editor: "<<this<<endl;
    #endif
     int i;
     for(i=0;i<=maxTerrenoBatalla;i++){
        #ifdef DEBUG
         cout << "Liberando Surface: "<<previews_niveles[i]<<endl;
        #endif
         SDL_FreeSurface(previews_niveles[i]);
     }
    #ifdef DEBUG
     cout << "Liberando Preview: "<<previews_niveles<<endl;
    #endif
    delete [] previews_niveles;
    delete data;
}


// explosion 

#include "explosion.hpp"


Explosion::Explosion(Juego * juego,int x,int y,int alcance_llamas,int lanzador,int id):
    Animacion(NULL,0,0,"0,1,2,3,4,4,3,2,1,1,0",x,y,id){
    this->juego=juego;
    
    this->lanzador=lanzador;
    this->alcance_llamas=alcance_llamas;
    detectarAlcance(DERECHA,16,0);
    detectarAlcance(IZQUIERDA,-16,0);
    detectarAlcance(ABAJO,0,16);
    detectarAlcance(ARRIBA,0,-16);

    //reproducimos un sonido
    juego->play(SFX_EXPLOSION);
    this->type=EXPLOSION;
}

void Explosion::detectarAlcance(int dir,int aum_x,int aum_y){

    int es_bloque_rompible,colision_con_bomba,colision_con_item;
    int globo_anyadido;
    SDL_Rect coli={0,0,16,16};
    for(int i=1;i<alcance_llamas+1;i++){
        coli.x=x+aum_x*i;
        coli.y=y+aum_y*i;
        alcances[dir]=i;
        if(coli.x<juego->getEjeXVisual()||\
           coli.x+coli.w>juego->getEjeXVisual()+juego->getAnchoMapa()||\
           coli.y<juego->getEjeYVisual()||\
           coli.y+coli.h>juego->getEjeYVisual()+juego->getAltoMapa()){
            --alcances[dir];
            return;
        }
        es_bloque_rompible=juego->isBloqueRompible(coli.x,coli.y);
        colision_con_bomba=juego->colision(BOMBA,coli);
        colision_con_item=juego->colision(ITEM,coli);

        if(colision_con_bomba!=-1){
            juego->setRepeticionBomba(colision_con_bomba,0);//se acelera el 'explote'
            --alcances[dir];
            return;
        }

        if(juego->isBloqueDuro(coli.x,coli.y)||es_bloque_rompible||colision_con_item!=-1){


            if(es_bloque_rompible&&juego->colision(BLOQUE,coli)==-1){
                juego->addSprite(BLOQUE,coli.x,coli.y);
            }else if(colision_con_item!=-1){
                if(juego->getTipoItem(colision_con_item)!=Item::ITEM_PUERTA){
                    juego->addSprite(new Animacion(juego->getImagen(IMG_ITEM_FIRE),1,7,"0,0,0,1,1,2,2,2,3,3,4,4,5,5,6,6",coli.x,coli.y));
                    juego->killSprite(ITEM,colision_con_item);
                }else{
                     int x,y;
                     juego->getPosicion(ITEM,colision_con_item,x,y);
//                     globo_anyadido=juego->addSprite(GLOBO,new Globox,y);
//                     if(globo_anyadido!=-1)
//                        juego->setProteccion(GLOBO,globo_anyadido,true);
                }
            }

            --alcances[dir];
            return;
        }

    }

}
void Explosion::dibujarFlama(int dir,int aum_x,int aum_y,int cuadro_normal,int cuadro_final,SDL_Surface * screen)
{
    int i;
	int x,y;

    for(i=1;i<alcances[dir];i++)
    {
        x=this->x+aum_x*i;
        y=this->y+aum_y*i;
        if(x+16>0&&x<W_SCREEN&&y<H_SCREEN&&y+16>0)
        imprimir_desde_grilla (juego->getImagen(IMG_EXPLOSION),cuadro_normal+COLUMNAS_EXPLO*getCuadro(),screen, x,y,FILAS_EXPLO,COLUMNAS_EXPLO,0);
    }
    x=this->x+aum_x*i;
    y=this->y+aum_y*i;
    if(x+10>0&&x<W_SCREEN&&y<H_SCREEN&&y+16>0){
        if(alcances[dir]==alcance_llamas)
        	imprimir_desde_grilla (juego->getImagen(IMG_EXPLOSION),cuadro_final+COLUMNAS_EXPLO*getCuadro(), screen,  x,y,FILAS_EXPLO,COLUMNAS_EXPLO,0);
        else if(alcances[dir])
        	imprimir_desde_grilla (juego->getImagen(IMG_EXPLOSION),cuadro_normal+COLUMNAS_EXPLO*getCuadro(),screen,  x,y,FILAS_EXPLO,COLUMNAS_EXPLO,0);
    }
}


void Explosion::draw(SDL_Surface * screen){

    //DIBUJAMOS EL CENTRO
    if(x+10>0&&x<W_SCREEN&&y<H_SCREEN&&y+16>0)
	     imprimir_desde_grilla (juego->getImagen(IMG_EXPLOSION),2+COLUMNAS_EXPLO*getCuadro(), screen, x,y,FILAS_EXPLO,COLUMNAS_EXPLO,0);

    dibujarFlama(DERECHA,16,0,3,4,screen);
    dibujarFlama(IZQUIERDA,-16,0,1,0,screen);
    dibujarFlama(ABAJO,0,16,6,7,screen);
    dibujarFlama(ARRIBA,0,-16,6,5,screen);


}

bool Explosion::colision(SDL_Rect & rect_coli){
    //COLISION CON EL EJE X de la bomba

    rect.x=x - alcances[IZQUIERDA]*16;
    rect.y=y;
    rect.w=alcances[IZQUIERDA]*16+16 + alcances[DERECHA]*16;
    rect.h=16;
    if(rects_colisionan(rect,rect_coli))
        return true;

    //COLISION CON EL EJE Y de la bomba
    rect.w=16;
    rect.x=x;
    rect.y=y - alcances[ARRIBA]*16;
    rect.h=alcances[ARRIBA]*16+16 + alcances[ABAJO]*16;
    if(rects_colisionan(rect,rect_coli))
        return true;

    return false;
     
}


// fuente imagen 

#include "fuente_imagen.hpp"

FuenteImagen::FuenteImagen(SDL_Surface *imagen,char orden[]){
    ima=imagen;
    strncpy(this->ordenLetras,orden,MAX_LETRAS_RECONOCIDAS);
    identificarRects();
}

/* 
 * Informa si la columna (parámetro 2) es una linea completa de pixeles
 * transparentes en la superficie ima (parámetro 1)
 *
 * Se asume que el color transparente es aquel que coincide con el pixel
 * situado en la posicion (0,0) de la superficie.
 */
bool FuenteImagen::esColumnaVacia(int columna){
	Uint32 pixel_transparente = get_pixel (ima, 0, 0);
	int fila;
	
	/* busca un pixel opaco */
	for (fila = 0; fila < ima->h; fila ++){
		if (pixel_transparente != get_pixel (ima, columna, fila))
			return false;
	}

	return true;
}

/*
 * Analiza la superficie ima buscando letras para registrarlas en el 
 * vector de rectángulos fuentes (segundo parámetro)
 *
 * Para analizar, el programa recorre la imágen de izquierda a derecha
 * 'leyendo' barras verticales de píxeles. Así determina donde comienza
 * y termina cada letra (debe existir una separación de, al menos, un 
 * pixel entre cada caracter)
 */
void FuenteImagen::identificarRects (){
	int indice_letra = 0;
	bool esta_recorriendo_letra = false;
	int w = 0; /* ancho de la letra actual */

	for (int i = 0; i < ima->w; i ++)
	{
		if (esColumnaVacia ( i))
		{
			
			/* si estaba leyendo una letra entonces encontró 
			 * el ancho de la misma */
			if (esta_recorriendo_letra)
			{
				fuentes[indice_letra].x=i - w;
                fuentes[indice_letra].y=0;
                fuentes[indice_letra].w=w;
                fuentes[indice_letra].h=ima->h;
				esta_recorriendo_letra = false;
				indice_letra ++;
			}
		}
		else
		{
			/* si está recorriendo una letra aumenta el contador
			 * de ancho, en caso contrario encontró donde comienza
			 * la siguiente letra
			 */
			if (esta_recorriendo_letra)
				w ++;
			else
			{
				w = 1;
				esta_recorriendo_letra = true;
			}
		}
	}

	cout<<"Econtré " <<indice_letra <<" letras en el archivo de fuentes\n";
}


/*
 * Relaciona un caracter con un número entero
 */
int FuenteImagen::obtener_indice (char caracter){
	int i;
	
	if (caracter == ' ')
		return 1;

	for (i = 0; ordenLetras [i]; i ++)
	{
		if (caracter == ordenLetras [i])
			return i;
	}
	
	cout <<"No se puede encontrar el índice de:"<<caracter<<endl;
	return -1;
}


/*
 * imprime un caracter sobre la superficie dst (generalmente screen)
 */
int FuenteImagen::imprimir_letra (SDL_Surface * dst,int x, int y, char letra){
	SDL_Rect srcrect;
	SDL_Rect dstrect = {x, y, 0, 0};
	int indice = obtener_indice (letra);

	if (letra == ' ')
		return fuentes [0].w;

	
	if (indice != -1)
	{
		srcrect = fuentes [indice];
		SDL_BlitSurface(ima, &srcrect, dst, &dstrect);
	}
	return srcrect.w;
}

/*
 * imprime una cadena de textos completa sobre la superficie referenciada
 * por el primer parámetro
 */
void FuenteImagen::draw (SDL_Surface * screen, int x, int y, char * cadena)
{
	int i;
	int dx = x;

	for (i = 0; cadena [i]; i ++)
		dx += imprimir_letra (screen, dx, y, cadena [i]);
}


// galeria 

#include "galeria.hpp"


Galeria::Galeria(){
    #ifdef DEBUG
    cout << "Constructor de galeria:"<<this<<endl;
    #endif
    
    char ruta[60];
    bool keyColor;
    
    ifstream filePar("data/configuracion/images.txt");
    if(!filePar)
        mostrar_error("No se pudo abrir archivo con RUTAS DE IMAGENES");
    
    for(int j=0;j<_IMAGENES;j++){
        filePar >>ruta;
        filePar >>keyColor;
        baulimgs[j]=cargar_imagen(ruta,keyColor);
    }
    filePar.close();
    sonidoCargado=false;
}

void Galeria::cargarSonidos(){
    char ruta_tmp[50];
    if(!sonidoCargado){
         for(int i=0;i<_SONIDOS;i++){
                sprintf(ruta_tmp,"data/sonidos/musica_%d.mid",i+1);
                snd_musicas[i]=cargar_musica(ruta_tmp);
            }
    
    
    
         for(int i=0;i<_EFECTOS;i++){
                sprintf(ruta_tmp,"data/sonidos/ping_%d.wav",i+1);
                sfx_efectos[i]=cargar_sonido(ruta_tmp);
                Mix_VolumeChunk(sfx_efectos[i], 100);
            }
    
        Mix_VolumeMusic(128);
        sonidoCargado=true;
    }
    
}

Galeria::~Galeria(){
    #ifdef DEBUG
    cout << "Destructor de galeria:"<<this<<endl;
    #endif

    for(int i=0;i<_IMAGENES;i++)SDL_FreeSurface(baulimgs[i]);
    if(sonidoCargado){
         for(int i=0;i<_SONIDOS;i++)Mix_FreeMusic(snd_musicas[i]);
         for(int i=0;i<_EFECTOS;i++)Mix_FreeChunk(sfx_efectos[i]);
    }
}

// gamer player 

#include "game_manager.hpp"

GameManager::GameManager(){
     /*inicializa la clase GameManager y la libreria*/
     srand(time(0));
     inter=NULL;
     interfaz_last=NULL;

     iniSDL();
     setModeVideo();
     
     activarJoysticks();/*solo se activan al inicio, si un joystick se conecta a la PC despues de haber corrido esta

     salir=false;
     /*creamos las clases en memoria*/
     galeria=new Galeria();
     if(snd_disponible)galeria->cargarSonidos();
     salir=false;

	// Initialize Frame Rate Manager
	SDL_initFramerate(&fpsm);
	SDL_setFramerate(&fpsm, 60); // 60 Frames Per Second
    SDL_WM_SetCaption("DestructionBombs",NULL);
//    sdl_videoinfo();
}
void GameManager::iniSDL(){
    if(!SDL_WasInit(SDL_INIT_VIDEO|SDL_INIT_AUDIO|SDL_INIT_JOYSTICK)){ 
       if(SDL_Init(SDL_INIT_VIDEO|SDL_INIT_AUDIO|SDL_INIT_JOYSTICK)<0){
           mostrar_error("No se pudo inciar SDL");
       }

        atexit(SDL_Quit);

        if(Mix_OpenAudio(MIX_DEFAULT_FREQUENCY, MIX_DEFAULT_FORMAT,MIX_DEFAULT_CHANNELS, 4096) < 0) {
            cerr << "[WARNING]%s" << SDL_GetError();
            snd_disponible=false;
        }else{
            snd_disponible=true;
            Mix_AllocateChannels(_PLAYERS + 1);
        }
        
    }

}

void GameManager::setModeVideo(bool pantalla_completa){
    int banderas=0;

    if(pantalla_completa)
        banderas|=SDL_FULLSCREEN;

    banderas |= SDL_DOUBLEBUF|SDL_SWSURFACE;
    
    SDL_Surface *icono;    
    icono =SDL_LoadBMP("data/imagenes/objetos/icono.bmp");
     // Establecemos el icono
    SDL_WM_SetIcon(icono, NULL); // Compatible con  Windows
    SDL_FreeSurface(icono);
    
    screen = SDL_SetVideoMode(W_SCREEN, H_SCREEN, 0, banderas);
    if(!screen){
        mostrar_error("No se pudo crear Frame-buffer");
    }

    SDL_WM_SetCaption("Cargando...",NULL);
    SDL_FillRect(screen,NULL,SDL_MapRGB(screen->format,104,104,104));

}

void GameManager::activarJoysticks(){
     /*contamos y abrimos los Joysticks que usara nuestro juego, Maximos 5 que pueden ser distintos*/
    joys_act=SDL_NumJoysticks();
     joys_act=(joys_act>5)?5:joys_act;//esto es porque solo podemos manejar 5...mas seria un error en este programa
     for(int i=0;i<joys_act;i++){
          joysticks[i]=SDL_JoystickOpen(i);//abrimos el joystick
     }
     for(int i=joys_act;i<_PLAYERS;i++)
          joysticks[i]=NULL;//los espacios que sobran los ponemos a NULL
}
SDL_Joystick * GameManager::getJoy(int id){
    return joysticks[id];
}
int GameManager::getJoysActivos(){
    return joys_act;
}

void GameManager::cambiarInterfaz(Interfaz *  nueva){
     /*cambia la interfaz del juego*/
    #ifdef DEBUG
        cout << "Cambiando interfaz a "<<nueva<<endl;
    #endif
//    cout << "1"<<endl;
    if(interfaz_last&&nueva!=interfaz_last){        
        #ifdef DEBUG
            cout << "Eliminando interfaz:"<<interfaz_last<<endl;
        #endif
        delete interfaz_last; //Para evitar una mala accion
        interfaz_last=NULL;
    }
//    cout << "Interfaz(ante la asignación):"<<inter<<" Nueva:"<<nueva<<endl;
    interfaz_last=inter;
//    cout << "1"<<endl;
    inter=nueva;
    #ifdef DEBUG
        cout << "Interfaz:"<<inter<<endl;
        cout << "Interfaz anterior:"<<interfaz_last<<endl;
    #endif
}

int GameManager::procesarEventos(){
    static bool full=1;//pantalla completa
    SDL_Event evento;
    while(SDL_PollEvent(&evento)){
        switch(evento.type){
            case SDL_QUIT:
                salir=1;
                return 0;
            break;
            case SDL_KEYDOWN:
                if((evento.key.keysym.sym==SDLK_RETURN && evento.key.keysym.mod & SDLK_LALT)||
                (evento.key.keysym.sym==SDLK_f && evento.key.keysym.mod & SDLK_LALT)){
                    setModeVideo(full);
                    SDL_WM_SetCaption("DestructionBombs",NULL);

                    full=!full;
                    return 0;
                }
                if(evento.key.keysym.sym==SDLK_F4 && evento.key.keysym.mod & SDLK_LALT){
                    salir=1;
                    return 0;
                }

            break;
        }
        
        inter->procesarEvento(&evento);
    }
    return 1;//se puede continuar
}


void GameManager::run(){
    if(!inter)return;
    while(!salir){
        if(procesarEventos()){
            inter->update();
            inter->draw(screen);
         }
        SDL_Flip(screen);
        SDL_framerateDelay(&fpsm);
    }

}

void GameManager::play(CodeMusicEfecto code){
    /*Reproduce un Chunk*/
    if(snd_disponible)Mix_PlayChannel(-1,galeria->getMusicEfecto(code), 0);
}
void GameManager::playSonido(CodeMusicSonido code){
    /*Reproduce una musica de fondo*/
    static int t_ini=0;
    static int t_pas=0;
    
    if(snd_disponible){
        if(SDL_GetTicks()-t_ini<1000){/*Si se reproduce este sonido seguidamente del anterior (1 s)*/
            cerr << "WARNING:Reproducción apresurada del sonido:"<<code<<endl;
        }
        Mix_PlayMusic(galeria->getMusicSonido(code), -1);
        t_ini=SDL_GetTicks();
    }
}

SDL_Surface * GameManager::getImagen(CodeImagen code){
    return galeria->getImagen(code);
}

GameManager::~GameManager(){
    #ifdef DEBUG
        cout << "Destructor de GameManager:"<<this<<endl;
    #endif

    delete inter;
    delete interfaz_last;
    delete galeria;

    for(int i=0;i<joys_act;i++)
        if(joysticks[i]&&SDL_JoystickOpened(i))
            SDL_JoystickClose(joysticks[i]);
            
    if(snd_disponible)
        Mix_CloseAudio();
    cout <<"Fin del juego... visita:http://baulprogramas.blogspot.com/\n";
}

/*
void GameManager::cargarDatos(){  
    ifstream fs(RUTA_CONFIG);
    if(!fs){
        cerr << "Error abriendo:--"<<RUTA_CONFIG<<endl;
        maxTerrenosBatalla=0;
        puntaje_mayor=0;
    }
    fs >>maxTerrenosBatalla;
    fs >>puntaje_mayor;
    fs.close();
}



void GameManager::guardarDatos(){
    ofstream fs(RUTA_CONFIG);
    fs <<maxTerrenosBatalla << endl;
    fs <<puntaje_mayor<<endl;
    fs.close();
}

int GameManager::getMaxTerrenoBatalla(){
    return maxTerrenosBatalla;
}
*/


/*SDL_Surface GameManager::getPreviewTerreno(int id,int id_tile){
     return Nivel::getPreviewTerreno(id,id_tile,galeria);
}

void GameManager::cargarFileNivel(char * buffer,char ruta[]){
     Nivel::cargarFileNivel(buffer,ruta);
}

void GameManager::dibujarNivel(char * mapa,int id_tile,SDL_Surface * super){
     Nivel::draw(mapa,galeria->tiles[id_tile],super);
*/

// globo 

/*#include "globo.h"
#include "objetos.h"
#include "nivel.h"
#include "player.h"
#include "engine/util.h"

Globo * globo_crear(){
    Globo * globo;
    globo=(Globo *)malloc(sizeof(Globo));
    if(globo==NULL)mostrar_error("Error asignando memoria a un globo");
    return globo;
}

void globo_iniciar(Globo * globo,Personajes *personajes,int id){
    globo->personajes=personajes;
    globo->en_pantalla=0;
	globo->rect_coli.w=13;//rectangulo que representa a nuestro personaje
	globo->rect_coli.h=10;//rectangulo que representa a nuestro personaje
    globo->id=id;
    globo->protegido=0;
}

void globo_activar(Globo * globo,int x, int y){
    static int estados_rand[4]={DERECHA,IZQUIERDA,ABAJO,ARRIBA};

    globo->delay_mover=0;
    globo->delay_para_morir=0;
    globo->paso=0;
    globo->delay_quitar_proteccion=0;
    globo->cuadro=0;
    globo->delay=0;
//    globo->vidas=1;
    globo->incremento_x=0;
    globo->incremento_y=0;
    globo->en_pantalla=1;
	globo->muerto=0;
    globo_set_coor(globo,x,y);
    globo_cambiar_estado(globo,estados_rand[rand()%4]);
 }
void globo_actualizar (Globo * globo)
{
    static const int estados_rand[4]={DERECHA,IZQUIERDA,ABAJO,ARRIBA};
    int colision_explo;

	globo_avanzar_animacion (globo);//avanzamos la animacion

    if(globo->estado!=MURIENDO){
        globo_actualizar_rect_colision(globo);

        if(!globo_mover (globo))
            globo_cambiar_estado(globo,estados_rand[rand()%4]);

        if(!globo->protegido){
            colision_explo=objetos_colision_con_explosiones(globo->personajes->juego->objetos,&globo->rect_coli,-1);
            if(colision_explo){
                globo_cambiar_estado (globo, MURIENDO);
                player_aumentar_puntaje(globo->personajes->players[globo->personajes->juego->objetos->explosiones[colision_explo-1]->lanzador],50);
            }
        }
        else if(++globo->delay_quitar_proteccion>DELAY_PROTECCCION){
            globo->protegido=0;
            globo->delay_quitar_proteccion=0;
        }

        }else if(globo->estado==MURIENDO&&globo->delay_para_morir++>100){
            globo_desactivar(globo);
            globo->delay_para_morir=0;
        }


}

int globo_colision(Globo * globo,SDL_Rect * rect_coli){
    globo_actualizar_rect_colision(globo);
    return rects_colisionan(&globo->rect_coli,rect_coli);
}

int globo_colision_con_otros(Globo * globo){
    register int i;
        /*este codigo se debe modificar si pongo un enemigo que no sea el "GLOBO" o sino no detectara la colision con el*/

	   /* for(i=0;i<globo->personajes->globos_actuales;i++){
            //basicamente lo que hace esto es comprobar colision con los globos de los alrededores
	       if((globo->personajes->globos[i]->en_pantalla)&&(i!=globo->id)){
                if(globo->personajes->globos[i]->estado!=MURIENDO&&globo_colision(globo->personajes->globos[i],&globo->rect_coli)){
                        return i+1;

                }//fin if colision con globo

            }//fin if globo activo
        }//fin for
    return 0;
}

int globo_mover (Globo * globo){
    int null=0; variable nula cuya unica funcion es pasarla como parametro a "niveles_colisiona" para no generar un error logico
    /*Estados estado_globo_colision;
    static const int estados_rand[4]={DERECHA,IZQUIERDA,ABAJO,ARRIBA};

    if(++globo->delay_mover>3){//para que el enemigo no se mueva a la misma velocidad que el personaje principal
        globo->delay_mover=0;
        globo->rect_coli.x+=globo->incremento_x;
        globo->rect_coli.y+=globo->incremento_y;


        if(nivel_colisiona(globo->personajes->juego->nivel,&globo->rect_coli,&null))
            return 0;

        null=globo_colision_con_otros(globo);
        if(null){
            estado_globo_colision=globo->personajes->globos[null-1]->estado;
            do
                globo_cambiar_estado(globo,estados_rand[rand()%4]);
            while(globo->estado==invertir_estado(estado_globo_colision));
        }

        if(objetos_colision_con_bombas(globo->personajes->juego->objetos,&globo->rect_coli))
            return 0;
        globo->x+=globo->incremento_x;
        globo->y+=globo->incremento_y;
    }
    return 1;

}

void globo_imprimir(Globo * globo,SDL_Surface * screen){
    	imprimir_desde_grilla (globo->personajes->juego->mundo->galeria->grilla_globo,globo->cuadro, screen, globo->x,globo->y,1, 6,globo->estado==MURIENDO||globo->protegido);


}

void globo_desactivar(Globo * globo){
    globo->en_pantalla=0;
	globo->muerto=1;
}

void globo_actualizar_rect_colision(Globo * globo){
    globo->rect_coli.x=globo->x+2;
    globo->rect_coli.y=globo->y+2;
}


void globo_avanzar_animacion (Globo * globo)
{
	static int animacion [7] = {0,1,2,3,4,5,-1};
	if (++globo->delay > 15){
		globo->delay = 0;
		if(animacion  [++globo->paso] == -1)
    		globo->paso = 0;
	}
	globo->cuadro = animacion[globo->paso];

}

void globo_cambiar_estado (Globo * globo, Estados nuevo){
	globo->estado = nuevo;
	switch(globo->estado){
        case DERECHA:
            globo->incremento_x=1;
            globo->incremento_y=0;
            break;
        case IZQUIERDA:
            globo->incremento_x=-1;
            globo->incremento_y=0;
            break;
        case ARRIBA:
            globo->incremento_x=0;
            globo->incremento_y=-1;
            break;
        case ABAJO:
            globo->incremento_x=0;
            globo->incremento_y=1;
            break;
        case MURIENDO:break;
        default:
            printf("error en estados del globo\n");
        }

}


void globo_set_coor(Globo * globo, int x, int y){
    globo->x=x;
    globo->y=y;
}*/

// item 

#include "item.hpp"

Item::Item(Juego * juego,int x,int  y, int tipo,int id):
    Animacion(NULL,0,0,"0,1",x,y,id){
    this->juego=juego;
    type=ITEM;
    loop=-1;
    
    if(!juego->getPuertaAbierta()&&juego->getTipoJuego()==TIPO_NORMAL&&y==juego->getYPuerta()&&x==juego->getXPuerta()){
        this->tipo=ITEM_PUERTA;
        juego->setPuertaAbierta(true);
        }
    else
        this->tipo=tipo;

}

void Item::draw(SDL_Surface * screen){
    if(x+16>0&&x<W_SCREEN&&y<H_SCREEN&&y+16>0)
    	imprimir_desde_grilla(juego->getImagen(IMG_ITEM), tipo/8*8 + tipo + 8*getCuadro(), screen, x,y,6,8,0);
}

// JUEGO 

#include "juego.hpp"

/*
*
*
*  FALTA AGREGAR UN BLOQUE EN LLAMAS
*
*
*/

Juego::Juego (GameManager * game){
    #ifdef DEBUG
    cout << "Constructor de Juego:"<<this<<endl;
    #endif
    this->game=game;

    sprites=new Group(this);
    data=NULL;
    clockTick=NULL;
    pausado=hold_start=false;
    
    id_quien_pauso=PLAYER_NONE;
    id_quien_pauso_anterior=PLAYER_NONE;
    
    estado=PLAY;
    _quit=false;
    estado_siguiente=NONE;
    SDL_ShowCursor(SDL_DISABLE);

    totalSprite[GLOBO]=20;
    totalSprite[EXPLOSION]=MAX_BOMBAS;
    totalSprite[ITEM]=30;
    totalSprite[BOMBA]=MAX_BOMBAS;
    totalSprite[PLAYER]=5;
    totalSprite[BLOQUE]=20;
    totalSprite[ANIMACION]=20;
}

void Juego::crearReferencias(){
    /*Inicia y crea las estructuras para administrar los personajes*/
    
    for(int i=0;i<_REFERENCIADOS;i++){
        refeSprites[i]=new Sprite* [totalSprite[i]];
        spriteActivos[i]=0;
        for(int j=0;j<totalSprite[i];j++)
            refeSprites[i][j]=NULL;
    }
    
}
void Juego::procesarEvento(SDL_Event * evento){
    switch(evento->type){
        case SDL_KEYDOWN:
             switch(evento->key.keysym.sym){
                case SDLK_ESCAPE:
                    salir();
                    break;
                case SDLK_TAB:
                     /*if(estado!=DISPLAY_MSG&&!pausado){
                         mostrar_kills=!mostrar_kills;
                         if(mostrar_kills)
                             juego_preparar_kills(juego);
                     }*/
                     break;
             }
        break;
        
    }
    
}

void Juego::displayMensage(const char *  mensage){
  //realiza operaciones para presentar un mensage al usuario

  estado=DISPLAY_MSG;
  y_msg=x_msg=0;
  vel_y=0.0;
  strncpy(msg_mostrar,mensage,50);
  estado_siguiente=PLAY;
}


void Juego::estadoDisplayMensage(){
//efecto sobre texto (autor: Hugo Ruscitii -http://www.loosersjuegos.com.ar- ) 
    static int dir=1;

    x_msg += dir;
	vel_y += 0.1;
	if (y_msg >= H_SCREEN/2&&vel_y>0)
	{
		vel_y -=1.2; // pierde fuerza al tocar el suelo
		vel_y *= -1;
        if((int)vel_y==0)
        {
            vel_y=0;
           estado=estado_siguiente;
//           retraso=50;
           if(_quit){
                salir();
            }
        }
	}
	else{
		y_msg += (int) vel_y;
	}
	if(x_msg > W_SCREEN-200)
	   dir=-1;
	else if(x_msg < 80)
	   dir=1;
}

void Juego::play(CodeMusicEfecto code) {
     game->play(code);
}
void Juego::playSonido(CodeMusicSonido code){
     game->playSonido(code);
}

int Juego::getActivos(TipoSprite type){
    return spriteActivos[type];
}

void Juego::clearSprites(bool elimina_players){
    /*Elimina todas las referencias de refeSprites si especifica "all_clear" sino no se borrara los players ni los cuadros en memoria*/
    #ifdef DEBUG
        cout << "Llamado a clearSprites"<<endl;
    #endif
     int count=0;
     for(int i=0;i<_REFERENCIADOS;i++){
        if(!elimina_players&&i==PLAYER)continue;
        
        if(spriteActivos[i]>0){
             for(int j=0;j<totalSprite[i];j++){
                 if(refeSprites[i][j]!=NULL){
                     sprites->erase(refeSprites[i][j]);
                    #ifdef DEBUG
                        cout << "Eliminando Sprite:"<<refeSprites[i][j]<<endl;
                    #endif
                     delete refeSprites[i][j];
                     refeSprites[i][j]=NULL;
                     if(++count==spriteActivos[i]){
                         spriteActivos[i]=0;
                         count=0;
                         break;
                     }
                }
             }
        }
     }
     
}

int Juego::getLanzador(TipoSprite type,int id_spri){
    if(type==BOMBA||type==EXPLOSION)
        return static_cast<Bomba *>((refeSprites[type][id_spri]))->getLanzador();
}

int Juego::getTipoItem(int id_item){
         return static_cast<Item *>(refeSprites[ITEM][id_item])->getTipoItem();
}

bool Juego::isActivo(TipoSprite type,int id){
     if(type==PLAYER)
         return refeSprites[type][id]!=NULL&&static_cast<Player *>(refeSprites[type][id])->isActivo();
     else
         return refeSprites[type][id]!=NULL;
}

void Juego::setPuntaje(IdPlayer id,int nuevo){
     static_cast<Player *>((refeSprites[PLAYER][id]))->setPuntaje(nuevo);
}

int Juego::getPuntaje(IdPlayer id){
     return static_cast<Player *>((refeSprites[PLAYER][id]))->getPuntaje();
}

void Juego::setEstadoPlayer(IdPlayer id,EstadoSprite nuevo){
     static_cast<Player *>((refeSprites[PLAYER][id]))->cambiarEstado(nuevo);
}

int Juego::getSegundosInicioNivel(){
    return clockTick->getTickInicial();
}
int Juego::getActivos(TipoSprite type,int & id){
    for(int i=0;i<totalSprite[type],spriteActivos[type];i++)
        if((refeSprites[type][i]!=NULL&&type!=PLAYER)||
        (type==PLAYER&&isActivo(PLAYER,i))){
             id=i;
             return spriteActivos[type];
        }
    return 0;
}

void Juego::setActivos(TipoSprite type,int nuevo){
    spriteActivos[type]=nuevo;
}

int Juego::getActivosId(TipoSprite type,IdPlayer  id){
    /*Obtiene todos los personajes activos cuyo lanzador fue el id*/
    int count=0;
    if(spriteActivos[type]){
        for(int i=0;i<totalSprite[type];i++){
            if(refeSprites[type][i]!=NULL&&(type==BOMBA||type==EXPLOSION)&&static_cast<Bomba *>(refeSprites[type][i])->getLanzador()==id){
                 count++;
            }
        }
    }
    return count;
}

void Juego::killSprite(int type,int id_sprite){
     if(refeSprites[type][id_sprite]!=NULL){
         refeSprites[type][id_sprite]->kill();
         spriteActivos[type]--;
         Sprite * spri=refeSprites[type][id_sprite]; // por si se necesitan saber datos delsprite que se eliminra antes de hacerlo
         refeSprites[type][id_sprite]=NULL; /*Notese que el grupo "sprites" sige referenciandolo, por lo que él
         se encargara de eliminarlo de memoria*/

         if(type==BOMBA){
             /*se agrega una explosion en la posicion de la bomba con el alcance que ella tenia*/
             addSprite(EXPLOSION,
                 spri->getX(),spri->getY(),
                 static_cast<Bomba *>(spri)->getLanzador(),static_cast<Bomba *>(spri)->getAlcance());
             
         }
         /*else if(type==BOMBA){
             /*se agrega una explosion en la posicion de la bomba con el alcance que ella tenia
             addSprite(EXPLOSION,
                 refeSprites[type][id_sprite]->getX(),refeSprites[type][id_sprite]->getY(),
                 refeSprites[type][id_sprite]->getLanzador(),refeSprites[type][id_sprite]->getAlcance());
             
         }*/
     }
     
}
void Juego::soloKill(int type,int id_sprite){
     if(refeSprites[type][id_sprite]!=NULL){
         refeSprites[type][id_sprite]->kill();
         spriteActivos[type]--;
         refeSprites[type][id_sprite]=NULL; /*Notese que el grupo "sprites" sige referenciandolo, por lo que él
         se encargara de eliminarlo de memoria*/
     }
     
}
void Juego::erase(int type,int id_sprite){
    //Elimina parcialmente a un sprite ya que no lo muestra pero sigue en memoria
     if(refeSprites[type][id_sprite]!=NULL){
         spriteActivos[type]--;
         sprites->erase(refeSprites[type][id_sprite]);
    }
}

void Juego::addSprite(Sprite * spri){
    int regresar;
    if(spriteActivos[spri->getTipo()]==totalSprite[spri->getTipo()]){
        delete spri;
        spri=NULL;
    }

    for(int i=0;i<totalSprite[spri->getTipo()];i++){
            if(refeSprites[spri->getTipo()][i]==NULL){
                refeSprites[spri->getTipo()][i]=spri;
                spriteActivos[spri->getTipo()]++;
                sprites->add(refeSprites[spri->getTipo()][i]);
                refeSprites[spri->getTipo()][i]->setId(i);
                return;
            }
    }
    return ;
}

int Juego::addSprite(int type,int x,int y,int lanzador,int otra_var){
    if(spriteActivos[type]==totalSprite[type]) return -1;//No hay espacio

    for(int i=0;i<totalSprite[type];i++){
            if(refeSprites[type][i]==NULL){
                if(type==BOMBA)
                    refeSprites[type][i]=new Bomba(getImagen(IMG_BOMBA),x,y,static_cast<Player *>(refeSprites[PLAYER][lanzador])->getAlcanceBombas(),(IdPlayer)lanzador,i);
                else if(type==EXPLOSION)
                    refeSprites[type][i]=new Explosion(this,x,y,otra_var,(IdPlayer)lanzador,i);/*otra_var=alcance*/    
                else if(type==ITEM)
                    refeSprites[type][i]=new Item(this,x,y,lanzador,i);/*lanzador=TipoItem*/    
                else if(type==BLOQUE)
                    refeSprites[type][i]=new Bloque(this,x,y,i);/*lanzador=TipoItem*/    
                else if(type==GLOBO)
//                    refeSprites[type][i]=new Explosion(this,x,y,otra_var,lanzador)/*otra_var=alcance*/  
                    cout << "Not implemented yet"<<endl;  

                spriteActivos[type]++;
                sprites->add(refeSprites[type][i]);
                return i;
            }
    }
    return -1;
                
}
void Juego::getPosicion(TipoSprite type, int id,int & x,int & y){
     x=refeSprites[type][id]->getX();
     y=refeSprites[type][id]->getY();
}


int Juego::colision(SDL_Rect & rect_coli,int * lado_colision,bool solo_bloques_duros){
    /*Comprueba si un rect colisiona con el nivel*/
    return mapa->colision(&rect_coli,lado_colision,solo_bloques_duros);
}

int Juego::colision(TipoSprite  type[],int tamanyo,SDL_Rect & rect_coli){
    /*Detecta colisiones por conjuntos de colision*/
    int id_coli;
    for(int i=0;i<tamanyo;i++){
        id_coli=colision(type[i],rect_coli);
        if(id_coli!=-1)return id_coli;
    }
    return -1;
}
int Juego::colision(TipoSprite type,SDL_Rect & rect_coli,int id_ignorar){
    /*regresa True si "rect_coli" colisiona con un Sprite de refeSprites[TYPE]*/
    if(type==NIVEL){
       int err=colision(rect_coli,&err,false);
       return (err)?1:-1;
    }else if(spriteActivos[type]>0){
        int cont=0;
	    for(int i=0;i<totalSprite[type];i++){
	       if(refeSprites[type][i]!=NULL&&i!=id_ignorar){
                    if(refeSprites[type][i]->colision(rect_coli)){
                            return i;//regresa el indice de con quien colisiona 
                    }
                if(++cont==spriteActivos[type])return -1;
            }
        }
    }
    return -1;
}

void Juego::update ()
{//actualiza la logica del juego
    switch(estado){
        case PLAY:
             if(!animandoEntradaMapaVertical)estadoPlay();
             break;
        case DISPLAY_MSG:
             estadoDisplayMensage();
             break;
        }
    if(animandoEntradaMapaVertical){
        desplazamiento+=2;
        mapa->setEjeVisualizacion(mapa->getEjeX(),H_SCREEN-desplazamiento);
        if(H_SCREEN-desplazamiento<=mapa->getEjeY()){
            animandoEntradaMapaVertical=false;
        }
    }

}
void Juego::draw(SDL_Surface * screen){
     
    dibujar_objeto(game->getImagen((CodeImagen)mapa->getIdFondo()),0,0,screen);
    drawBarra(screen);//imprimimos la barra mensage
    mapa->draw(screen);//imprimimos el nivel
    sprites->draw(screen);

    if(pausado){
        imprimir_palabra(screen, game->getImagen(IMG_FUENTE_5),80,100,"pausa",STR_NORMAL);
        imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN),id_quien_pauso*2 ,screen,130,90,1,10,0);
    }
    if(estado==DISPLAY_MSG)imprimir_palabra (screen,game->getImagen(IMG_FUENTE_6),x_msg,y_msg,msg_mostrar,STR_MAX_ESTENDIDA);
}

void Juego::setProteccion(TipoSprite type, int id,int nuevo){
     if(type==GLOBO||type==PLAYER){
         static_cast<Player *>(refeSprites[type][id])->setProteccion(nuevo);
     }
}

void Juego::setRepeticionBomba(int id,int nuevo){
     if(refeSprites[BOMBA][id]!=NULL)
         static_cast<Bomba *>(refeSprites[BOMBA][id])->setRepeticion(nuevo);
}
bool Juego::isBloqueRompible(int x,int y){
     return mapa->isBloqueRompible(x,y);
}
bool Juego::isBloqueDuro(int x,int y){
    return mapa->getTipoBloque(x,y)==Mapa::BLOQUE_METAL;
}
bool Juego::getPuertaAbierta(){
     return p_abierta;
}

bool Juego::isBloqueItem(int x,int y){
     return mapa->getTipoBloque(x,y)==Mapa::BLOQUE_ITEM;
}

void Juego::setPuertaAbierta(bool nuevo){
     p_abierta=nuevo;
}

void Juego::romperBloque(int x,int y){
    mapa->romperBloque(x,y);
}

int Juego::getJoysActivos(){
    return game->getJoysActivos();
}
SDL_Joystick * Juego::getJoy(int id){
    return game->getJoy(id);
}

void Juego::controlaPausa(Uint8 * teclas){
    /*SI ALGUN PLAYER PRESIONO START SE PAUSA EL JUEGO*/
    if(!hold_start){
        for(int i=0;i<totalSprite[PLAYER];i++){
            if(isActivo(PLAYER,i)&&\
                static_cast<Player *>(refeSprites[PLAYER][i])->isPressed(TECLA_START,teclas)){
                if(!pausado){
                    pausado=true;
                    inicio_pausa=time(0);
                    id_quien_pauso=(IdPlayer)i;
                    play(SFX_CAMPANADA);
                }else if(id_quien_pauso==(IdPlayer)i){
					pausado=false;
					clockTick->setTicksPerdidos(clockTick->getTickPerdidos()+time(0)-inicio_pausa);
					id_quien_pauso_anterior=id_quien_pauso; //se guarda el player que habia puesto pausa, para que no mantenga START presionada
					id_quien_pauso=PLAYER_NONE;
					play(SFX_TONO_ACUATICO);
                    
                }
                break;
            }
        }
    }

	if(id_quien_pauso!=PLAYER_NONE)
		hold_start=static_cast<Player *>(refeSprites[PLAYER][id_quien_pauso])->isPressed(TECLA_START,teclas);
	else if(id_quien_pauso_anterior!=PLAYER_NONE)
		hold_start=static_cast<Player *>(refeSprites[PLAYER][id_quien_pauso_anterior])->isPressed(TECLA_START,teclas);
}

int Juego::getTipoNuevoItem(bool disminuir_de_mapa){

    static int tipos[5]={Item::ITEM_BOLA_ARROZ,
                         Item::ITEM_PASTEL,
                         Item::ITEM_PALETA,
                         Item::ITEM_BARQUILLA,
                         Item::ITEM_MANZANA};

    int tmp=rand()%1200,indice;
    if(tmp<10&&tmp>=0)indice=Item::ITEM_BOMBA_MAX; //0.8% probabilidades de aparecer
    else if(tmp<20&&tmp>=10)indice=Item::ITEM_ALCANCE_MAX;//0.8%
    else if(tmp<120&&tmp>=20)indice=Item::ITEM_ALCANCE; //8% 
    else if(tmp<180&&tmp>=120)indice=Item::ITEM_VIDA; //4% 
    else if(tmp<280&&tmp>=180)indice=Item::ITEM_BOMBA;//8% 
    else if(tmp<330&&tmp>=280)indice=Item::ITEM_ATRAVIESA_PAREDES;//4% 
    else if(tmp<430&&tmp>=330)indice=Item::ITEM_PROTECCION;//8%
    else if(tmp<530&&tmp>=430)indice=Item::ITEM_ALEATORIO;//8%
    else if(tmp<580&&tmp>=530)indice=Item::ITEM_PATINETA; //4%
    else if(tmp<630&&tmp>=580)indice=Item::ITEM_CORAZON; //4% 
    else if(tmp<680&&tmp>=630)indice=Item::ITEM_ATRAVIESA_BOMBAS;//4%
    else indice=tipos[rand() % 5 ];//54%
    
    if(disminuir_de_mapa&&mapa->getNumItems()>=1)
            mapa->setNumItems(mapa->getNumItems()-1); 
    
    return indice;
}

void Juego::moveAllSprites(int aumX,int aumY){
     int count=0;
     for(int i=0;i<_REFERENCIADOS;i++){
        if(spriteActivos[i]>0){
             for(int j=0;j<totalSprite[i];j++){
                 if(refeSprites[i][j]!=NULL){
                     refeSprites[i][j]->setX(refeSprites[i][j]->getX()+aumX);
                     refeSprites[i][j]->setY(refeSprites[i][j]->getY()+aumY);
                     if(++count==spriteActivos[i]){
                         count=0;
                         break;
                     }
                }
             }
        }
     }
}

Juego::~Juego(){
        #ifdef DEBUG
        cout << "Destructor de Juego:"<<this<<endl;
        #endif

     for(int i=0;i<_REFERENCIADOS;i++){
        for(int j=0;j<totalSprite[i];j++){
            if(refeSprites[i][j]!=NULL){
                delete refeSprites[i][j];
                refeSprites[i][j]=NULL;
                }
        }
        delete [] refeSprites[i];
     }
        delete sprites;
        delete mapa;
        delete data;
        delete clockTick;
}

//  JUEGO BATALLA 

#include "juego_batalla.hpp"


JuegoBatalla::JuegoBatalla (GameManager * game,int idTerrenoBatalla,bool playerEnBatalla[_PLAYERS],int minutos,int victorias):Juego(game){
    #ifdef DEBUG
    cout << "Constructor de JuegoBatalla:"<<this<<endl;
    #endif

    EjeX=27;
    EjeY=54;
    mapa=new Mapa(this,EjeX,EjeY);

    totalSprite[GLOBO]=0;
    crearReferencias();
    
    this->idTerrenoActual=idTerrenoBatalla;
    setMapaPlay(idTerrenoBatalla,false);
    
    Player * player_;
    for(int i=0;i<_PLAYERS;i++){
        batallasGanadas[i]=0;
        if(playerEnBatalla[i]){
             spriteActivos[PLAYER]++;
             player_=new Player(this,(IdPlayer)i,\
                                      data->getX(i),\
                                      data->getY(i),\
                                      data->getVidas(),\
                                      data->getBombas(),\
                                      data->getAlcanceBombas());
            refeSprites[PLAYER][i]=player_;
            sprites->add(player_);
        }
    }
    
    displayMensage("¡Qué gane el mejor!");//se presenta el mensage con el nivel actual
//    estado_siguiente=PLAY;
    id_lider_ganadas=PLAYER_NONE;
    SDL_ShowCursor(SDL_DISABLE);
    min=minutos*60;
    vic=victorias;
    mapa->setImgTiles(game->getImagen(IMG_TILES));
}

void JuegoBatalla::crearReferencias(){
    for(int i=0;i<_REFERENCIADOS;i++){
        if(totalSprite[i]){
            refeSprites[i]=new Sprite* [totalSprite[i]];
            spriteActivos[i]=0;
            for(int j=0;j<totalSprite[i];j++){
                refeSprites[i][j]=NULL;
            }
        }
    }
}

void JuegoBatalla::setMapaPlay(int idTerreno,bool comprobar_players){
    char ruta[30],ruta2[50];
    sprintf(ruta,"data/niveles/batalla/%d.txt",idTerreno + 1);    
    sprintf(ruta2,"data/niveles/batalla/%d.map",idTerreno + 1);
    if(data)delete data;
    data=new DatNivel(ruta);
    if(clockTick)delete clockTick;
    clockTick=new TimeController();
    clockTick->setTicksPerdidos(4);
    
    if(comprobar_players){
        for(int i=0;i<_PLAYERS;i++){
            if(refeSprites[PLAYER][i]){//si fue elegido para que batalle
                if(!isActivo(PLAYER,i)){//si no esta en pantalla
                    sprites->add(refeSprites[PLAYER][i]);
                    spriteActivos[PLAYER]++;
                }/*else{//si sobrevivio a la batalla
                    static_cast<Player *>(refeSprites[PLAYER][i])->posicionInicial();
                }*/
                static_cast<Player *>(refeSprites[PLAYER][i])->setVidas(data->getVidas());
                static_cast<Player *>(refeSprites[PLAYER][i])->reiniciar();
            }
        }
    }

    clearSprites();
    mapa->cargarDeArchivoBin(ruta2,ruta);
    mapa->setEjeVisualizacion(mapa->getEjeX(),H_SCREEN);
    mapa->setItems();

    playSonido((CodeMusicSonido)(4 + rand()%1));
    muertosPorTiempo=false;
    repro_war=false;
    iniciado=false;
    patinesLanzados=false;
    bombasMaxLanzado=false;
    alcanceMaxLanzado=false;
    animandoEntradaMapaVertical=true;
    desplazamiento=0;
}

int JuegoBatalla::getTipoNuevoItem(bool hacerComprobaciones){
    int tmp,indice=-1;
    bool aprobado=false;
    do{
        tmp=rand()%1200;
        if(tmp<10&&tmp>=0&&!bombasMaxLanzado){//0.8% probabilidades de aparecer
            indice=Item::ITEM_BOMBA_MAX; 
            bombasMaxLanzado=true;
        }
        else if(tmp<20&&tmp>=10&&!alcanceMaxLanzado){//0.8%
            indice=Item::ITEM_ALCANCE_MAX; 
            alcanceMaxLanzado=true;
        }
        else if(tmp<120&&tmp>=20)indice=Item::ITEM_ALCANCE; //8% 
        else if(tmp<180&&tmp>=120)indice=Item::ITEM_BOMBA; //4% 
        else if(tmp<280&&tmp>=180)indice=Item::ITEM_ATRAVIESA_PAREDES;//8% 
        else if(tmp<330&&tmp>=280)indice=Item::ITEM_PROTECCION;//4% 
        else if(tmp<430&&tmp>=330)indice=Item::ITEM_ALEATORIO;//8%
        else if(tmp<480&&tmp>=430)indice=Item::ITEM_CORAZON;//4%
        else if(tmp<530&&tmp>=480)indice=Item::ITEM_ATRAVIESA_BOMBAS;//4%
        else if(tmp<580&&tmp>=530&&!patinesLanzados){//4% 
            indice=Item::ITEM_PATINETA; 
            patinesLanzados=true;
        }
        if(tmp>=0&&tmp<580&&indice!=-1)aprobado=true;
        
    }while(!aprobado);

    if(hacerComprobaciones&&mapa->getNumItems()>=1){
            mapa->setNumItems(mapa->getNumItems()-1); 
    }
    return indice;
}

void JuegoBatalla::estadoPlay(){
    Uint8 * teclas=SDL_GetKeyState (NULL);//se obtiene el estado actual del teclado

    if(!pausado){
            sprites->update(teclas);
            clockTick->update();            
            /*SI SE ACABO EL TIEMPO*/
            if(!muertosPorTiempo&&clockTick->getMiliSegundos()>=min){ 
                for(int i=0;i<_PLAYERS;i++){
                    if(refeSprites[PLAYER][i]&&isActivo(PLAYER,i)){
                        static_cast<Player *>(refeSprites[PLAYER][i])->cambiarEstado(MURIENDO);
                        static_cast<Player *>(refeSprites[PLAYER][i])->setVidas(0);
                    }
                }
                muertosPorTiempo=true;
            }
                
            /*SI SE ACERCA EL TIEMPO PARA ACABAR*/
            if(clockTick->getMiliSegundos()>min/3&&!repro_war){
                playSonido(SND_WARNING_TIME);
                repro_war=true;
            }

            int id_activo=-1;
            int total_activos=0;

            total_activos=getActivos(PLAYER,id_activo);

            if(total_activos==1){
                static char msg_ganador[20];
                batallasGanadas[id_activo]++;
                if((id_lider_ganadas==PLAYER_NONE)||(id_lider_ganadas!=PLAYER_NONE&&batallasGanadas[id_activo] > batallasGanadas[id_lider_ganadas]))
                        id_lider_ganadas=(IdPlayer)id_activo;
                else if(id_lider_ganadas!=PLAYER_NONE&&id_lider_ganadas!=id_activo&&batallasGanadas[id_activo] == batallasGanadas[id_lider_ganadas])
                        id_lider_ganadas=PLAYER_NONE;
                game->cambiarInterfaz(new JuegoMostrarGanadas(game,this,batallasGanadas));
                
                if(batallasGanadas[id_activo]>=vic){
                    sprintf(msg_ganador,"¡PLAYER %d GANÓ!",id_activo+1);
                    displayMensage(msg_ganador);
                    _quit=true;
                }else{
                    setMapaPlay(idTerrenoActual);
                }
            }else if(total_activos==0){
                char msg_ganador[20];
                sprintf(msg_ganador,"empate");
                displayMensage(msg_ganador);
                setMapaPlay(idTerrenoActual);
            }

      
    }//fin if(!pausado)
    
    controlaPausa(teclas);
}
void JuegoBatalla::salir(){
    _quit=true;
    game->cambiarInterfaz(new Menu(game));

}

void JuegoBatalla::drawBarra(SDL_Surface * screen){
    char tmp[50];
    
    dibujar_objeto(game->getImagen(IMG_TABLERO),0,mapa->getYPanel(),screen);

    //PLAYER_1
    imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN), !(refeSprites[PLAYER][PLAYER_1]&&isActivo(PLAYER,PLAYER_1)) + PLAYER_1*2 ,screen,1,24,1,10,0);

    dibujar_objeto(game->getImagen(IMG_CUADRO_PEQUENIO),15,21,screen);

    /*DIBUJAMOS LAS VIDAS RESTANTES*/
    if(refeSprites[PLAYER][PLAYER_1]&&isActivo(PLAYER,PLAYER_1)){
        sprintf(tmp,"%d",static_cast<Player *>(refeSprites[PLAYER][PLAYER_1])->getVidas());
    	imprimir_palabra (screen,game->getImagen(IMG_FUENTE_3),15,24,tmp,STR_ESTENDIDA);
    }

    //PLAYER_2
    imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN), !(refeSprites[PLAYER][PLAYER_2]&&isActivo(PLAYER,PLAYER_2)) + PLAYER_2*2 ,screen,32,24,1,10,0);

    dibujar_objeto(game->getImagen(IMG_CUADRO_PEQUENIO),48,21,screen);

    /*DIBUJAMOS LAS VIDAS RESTANTES*/
    if(refeSprites[PLAYER][PLAYER_2]&&isActivo(PLAYER,PLAYER_2)){
        sprintf(tmp,"%d",static_cast<Player *>(refeSprites[PLAYER][PLAYER_2])->getVidas());
    	imprimir_palabra (screen,game->getImagen(IMG_FUENTE_3),48,24,tmp,STR_ESTENDIDA);
    }

    //PLAYER_3
    imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN), !(refeSprites[PLAYER][PLAYER_3]&&isActivo(PLAYER,PLAYER_3)) + PLAYER_3*2 ,screen,65,24,1,10,0);

    dibujar_objeto(game->getImagen(IMG_CUADRO_PEQUENIO),80,21,screen);

    /*DIBUJAMOS LAS VIDAS RESTANTES*/
    if(refeSprites[PLAYER][PLAYER_3]&&isActivo(PLAYER,PLAYER_3)){
        sprintf(tmp,"%d",static_cast<Player *>(refeSprites[PLAYER][PLAYER_3])->getVidas());
    	imprimir_palabra (screen,game->getImagen(IMG_FUENTE_3),80,24,tmp,STR_ESTENDIDA);
    }

    //PLAYER_4
    imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN), !(refeSprites[PLAYER][PLAYER_4]&&isActivo(PLAYER,PLAYER_4)) + PLAYER_4*2 ,screen,253,24,1,10,0);

    dibujar_objeto(game->getImagen(IMG_CUADRO_PEQUENIO),270,21,screen);

    /*DIBUJAMOS LAS VIDAS RESTANTES*/
    if(refeSprites[PLAYER][PLAYER_4]&&isActivo(PLAYER,PLAYER_4)){
        sprintf(tmp,"%d",static_cast<Player *>(refeSprites[PLAYER][PLAYER_4])->getVidas());
    	imprimir_palabra (screen,game->getImagen(IMG_FUENTE_3),271,23,tmp,STR_ESTENDIDA);
    }

    //PLAYER_5
    imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN), !(refeSprites[PLAYER][PLAYER_5]&&isActivo(PLAYER,PLAYER_5)) + PLAYER_5*2 ,screen,288,24,1,10,0);

    dibujar_objeto(game->getImagen(IMG_CUADRO_PEQUENIO),304,21,screen);

    /*DIBUJAMOS LAS VIDAS RESTANTES*/
    if(refeSprites[PLAYER][PLAYER_5]&&isActivo(PLAYER,PLAYER_5)){
        sprintf(tmp,"%d",static_cast<Player *>(refeSprites[PLAYER][PLAYER_5])->getVidas());
    	imprimir_palabra (screen,game->getImagen(IMG_FUENTE_3),305,23,tmp,STR_ESTENDIDA);
    }

    if(id_lider_ganadas!=PLAYER_NONE)
        imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN_GRANDES),id_lider_ganadas,screen,154,-10,1,5,0);
    
    dibujar_objeto(game->getImagen(IMG_CUADRO_GRANDE),137,21,screen);
    
    if(clockTick){
        static char min_[3],seg[3],tiempo[6];
    
        sprintf(min_,"%2d",(min-clockTick->getMiliSegundos())/60);
        if(min_[0]==' ')min_[0]='0';
        sprintf(seg,"%2d",min-clockTick->getMiliSegundos()-(min-clockTick->getMiliSegundos())/60*60);
        if(seg[0]==' ')seg[0]='0';
        sprintf(tiempo,"%s:%s",min_,seg);
    
    	imprimir_palabra (screen,game->getImagen(IMG_FUENTE_6),\
    						142,24,tiempo,STR_MAX_ESTENDIDA);
    }
}
JuegoBatalla::~JuegoBatalla(){
    #ifdef DEBUG
        cout << "Destructor de JuegoBatalla:"<<this<<endl;
    #endif
}

/*
    cout << "1.1.1"<<endl;
void juego_dibujar_kills(Juego * juego,SDL_Surface * screen){
     char tmp[50];
    dibujar_objeto(game->galeria->tabs_kills,0,0,screen);
    int i;
    
    for(i=0;i<_PLAYERS;i++){
        dibujamos el primer player
        imprimir_desde_grilla(game->galeria->grilla_caras_bomberman,!(i==0) + sort_kills[i]*2 ,screen,39,67 + 29*i,1,10,0);
        dibujamos el player
        sprintf(tmp,"player %d",sort_kills[i]+1);
       	imprimir_palabra (screen, game->galeria->grilla_fuente_1,58,69 + 28*i,tmp,STR_NORMAL);
       	
        numero de matadas
        sprintf(tmp,"%d",matadas[sort_kills[i]]);
       	imprimir_palabra (screen, game->galeria->grilla_fuente_3,174,69 + 28*i,tmp,STR_ESTENDIDA);
     	
        numero de kills
        sprintf(tmp,"%d",kills[sort_kills[i]]);
       	imprimir_palabra (screen, game->galeria->grilla_fuente_3,248,68 + 28*i,tmp,STR_ESTENDIDA);
    }

}




void juego_preparar_kills(Juego * juego){
     mostrar_kills=1;
     sort_array(matadas,sort_kills);
}
*/

// JUEGO HISTORIA 

#include "juego_historia.hpp"

JuegoHistoria::JuegoHistoria (GameManager * game):Juego(game){
    /*Inicializa las referencias y pone todo para que se inicialize el juego*/
    #ifdef DEBUG
    cout << "Constructor de JuegoHistoria:"<<this<<endl;
    #endif

    EjeX=0;
    EjeY=0;
    mapa=new Mapa(this);
    mapa->setImgTiles(game->getImagen(IMG_TILES ));

    totalSprite[PLAYER]=2;
    crearReferencias();
    refeSprites[PLAYER][PLAYER_1]=new Player(this,PLAYER_1,X_INIT_PLAYER_1,Y_INIT_PLAYER_1,3,3);
    if(!refeSprites[PLAYER][PLAYER_1])
        cout << "Warning--No se pudo crear el Player 1"<<endl;
    refeSprites[PLAYER][PLAYER_2]=new Player(this,PLAYER_2,X_INIT_PLAYER_2,Y_INIT_PLAYER_2);
    if(!refeSprites[PLAYER][PLAYER_2])
        cout << "Warning--No se pudo crear el Player 2"<<endl;
    static_cast<Player *>(refeSprites[PLAYER][PLAYER_2])->setEnPantalla(false); /*Desactivamos el player2*/
    spriteActivos[PLAYER]=1; /*Solo estará activo el player 1*/
    sprites->add(refeSprites[PLAYER][PLAYER_1]); /*Lo agregamos al Grupo*/
    mayor_puntaje=0;
    setMapaPlay(1,false); /*Se empezara a jugar desde el Nivel 1*/
    static_cast<Player *>(refeSprites[PLAYER][PLAYER_2])->disable(); /*Desactivamos el player2*/
    
}

void JuegoHistoria::setMapaPlay(int id_nivel,bool reiniciar_jugadores){
    char ruta1[50],ruta2[50],ruta3[50];

    if(id_nivel<1)id_nivel=1;

    if(id_nivel<MAX_NIVEL){
        if(clockTick)delete clockTick;
        clockTick=new TimeController();
        clockTick->setTicksPerdidos(4);

        clearSprites();
        for(int i=PLAYER_1;i<=PLAYER_2;i++){
            if(isActivo(PLAYER,i)){
                if(reiniciar_jugadores){
                    static_cast<Player *>(refeSprites[PLAYER][i])->reiniciar();
                }else{
                    static_cast<Player *>(refeSprites[PLAYER][i])->posicionInicial();
                    static_cast<Player *>(refeSprites[PLAYER][i])->setProteccion(10);
                }
            }
        }
    
        sprintf(ruta1,"data/niveles/historia/nivel_%d.map",id_nivel);
        sprintf(ruta2,"data/niveles/historia/%d.txt",id_nivel);
        mapa->cargarDeArchivoBin(ruta1,ruta2);
        mapa->setEjeVisualizacion(mapa->getEjeX(),mapa->getEjeY());
        mapa->setItems();
        mapa->setEnemigos();
        char mens[50];
        sprintf(mens,"NIVEL %d",id_nivel);
        displayMensage(mens);//se presenta el mensage con el nivel actual
        repro_war=false; 
        playSonido((CodeMusicSonido)(4 + rand() % 1));
        n_actual=id_nivel;
        p_abierta=false;
        patinesLanzados=false;
        bombasMaxLanzado=false;
        alcanceMaxLanzado=false;
    }else{
        displayMensage("Te terminaste el juego :P, ¡FELICIDADES!");//se presenta el mensage con el nivel actual
        _quit=true;
    }
}

void JuegoHistoria::aumentarNivel(){
    int bonus=time(0)-getSegundosInicioNivel();

    for(int i=0;i<=PLAYER_2;i++){
        if(isActivo(PLAYER,i)){
            static_cast<Player *>(refeSprites[PLAYER][i])->setPuntaje(static_cast<Player *>(refeSprites[PLAYER][i])->getPuntaje()+bonus);
            static_cast<Player *>(refeSprites[PLAYER][i])->setEntroPuerta(false);
            static_cast<Player *>(refeSprites[PLAYER][i])->setProteccion(10);
            static_cast<Player *>(refeSprites[PLAYER][i])->cambiarEstado(PARADO);
        }
    }
    setMapaPlay(n_actual + 1,false);
}



void JuegoHistoria::estadoPlay(){
    Uint8 * teclas=SDL_GetKeyState (NULL);//se obtiene el estado actual del teclado
    if(!pausado){
        sprites->update(teclas);
            
        /*SI NO HAY PLAYERS ACTIVOS*/
        if(!isActivo(PLAYER,PLAYER_1) && !isActivo(PLAYER,PLAYER_2)){
            displayMensage("game over");
           _quit=true;
        }
        
         clockTick->update();
        /*SI SE ACABO EL TIEMPO*/
        if(clockTick->getMiliSegundos()>=_TIME_POR_NIVEL) 
            setMapaPlay(n_actual,true);    
            
        /*SI SE ACERCA EL TIEMPO PARA ACABAR*/
        if(clockTick->getMiliSegundos()==_TIME_POR_NIVEL-50&&!repro_war){
            playSonido(SND_WARNING_TIME);
            repro_war=true;
        }
        
        /*SI ALGUN PLAYER PRESIONO START Y ESTABA MUERTO*/
        for(int i=0;i<=PLAYER_2;i++)
            if(!isActivo(PLAYER,i)&&static_cast<Player *>(refeSprites[PLAYER][i])->isPressed(TECLA_START,teclas)){
                if(static_cast<Player *>(refeSprites[PLAYER][!i])->getVidas()>0){
                    static_cast<Player *>(refeSprites[PLAYER][!i])->setVidas(static_cast<Player *>(refeSprites[PLAYER][!i])->getVidas()-1);
                    static_cast<Player *>(refeSprites[PLAYER][i])->reiniciar();
                    static_cast<Player *>(refeSprites[PLAYER][i])->setVidas(0);
                    hold_start=true;//controlamos que no mantenga la tecla presionada
                    sprites->add(refeSprites[PLAYER][i]);
                    break;
                }
            }
    }
    controlaPausa(teclas);

}

inline void JuegoHistoria::salir(){
    _quit=true;
    game->cambiarInterfaz(new Menu(game));
}


void JuegoHistoria::drawBarra(SDL_Surface * screen){
    char tmp[50];

    dibujar_objeto(game->getImagen(IMG_TABLERO),0,mapa->getYPanel(),screen);
    
    
    //Dibujamos el tiempo
    dibujar_objeto(game->getImagen(IMG_CUADRO_GRANDE),129,3+mapa->getYPanel(),screen);
    if(clockTick){
        static char min[3],seg[3],tiempo[6];
    
        sprintf(min,"%2d",(_TIME_POR_NIVEL-clockTick->getMiliSegundos())/60);
        if(min[0]==' ')min[0]='0';
        sprintf(seg,"%2d",_TIME_POR_NIVEL-clockTick->getMiliSegundos()-(_TIME_POR_NIVEL-clockTick->getMiliSegundos())/60*60);
        if(seg[0]==' ')seg[0]='0';
        sprintf(tiempo,"%s:%s",min,seg);
    
    	imprimir_palabra (screen,game->getImagen(IMG_FUENTE_6),\
    						136,6+mapa->getYPanel(),tiempo,STR_MAX_ESTENDIDA);
    }
    
    if(isActivo(PLAYER,PLAYER_1)){
        
        //DIBUJAMOS LAS VIDAS
        dibujar_objeto(game->getImagen(IMG_CUADRO_PEQUENIO),59,5+mapa->getYPanel(),screen);
        sprintf(tmp,"%d",static_cast<Player *>(refeSprites[PLAYER][PLAYER_1])->getVidas());
    	imprimir_palabra (screen,game->getImagen(IMG_FUENTE_1),59,8+mapa->getYPanel(),tmp,STR_NORMAL);
        
        //Dibujamos la cara
        imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN_GRANDES),PLAYER_1,screen,28,-8+mapa->getYPanel(),1,5,0);

        //DIBUJAMOS EL PUNTAJE
        dibujar_objeto(game->getImagen(IMG_CUADRO_MEDIANO),4,23+mapa->getYPanel(),screen);
        sprintf(tmp,"%d",static_cast<Player *>(refeSprites[PLAYER][PLAYER_1])->getPuntaje());
        imprimir_palabra (screen,game->getImagen(IMG_FUENTE_2),40,24+mapa->getYPanel(),tmp,STR_NORMAL);
    }else{
        //Dibujamos la cara
        imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN_GRANDES),PLAYER_1,screen,28,-8+mapa->getYPanel(),1,5,0);

        dibujar_objeto(game->getImagen(IMG_TXT_PRESIONA_START),5,27+mapa->getYPanel(),screen);
    }

    if(isActivo(PLAYER,PLAYER_2)){
        
        //DIBUJAMOS LAS VIDAS
        dibujar_objeto(game->getImagen(IMG_CUADRO_PEQUENIO),292,5+mapa->getYPanel(),screen);
        sprintf(tmp,"%d",static_cast<Player *>(refeSprites[PLAYER][PLAYER_2])->getVidas());
    	imprimir_palabra (screen,game->getImagen(IMG_FUENTE_1),293,8+mapa->getYPanel(),tmp,STR_NORMAL);
        
        //Dibujamos la cara
        imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN_GRANDES),PLAYER_2,screen,262,-8+mapa->getYPanel(),1,5,0);

        //DIBUJAMOS EL PUNTAJE
        dibujar_objeto(game->getImagen(IMG_CUADRO_MEDIANO),232,23+mapa->getYPanel(),screen);
        sprintf(tmp,"%d",static_cast<Player *>(refeSprites[PLAYER][PLAYER_2])->getPuntaje());
        imprimir_palabra (screen,game->getImagen(IMG_FUENTE_2),273,24+mapa->getYPanel(),tmp,STR_NORMAL);

    }else{
        //Dibujamos la cara
        imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN_GRANDES),PLAYER_2,screen,262,-8+mapa->getYPanel(),1,5,0);

        dibujar_objeto(game->getImagen(IMG_TXT_PRESIONA_START),225,21+mapa->getYPanel(),screen);
    }
}

JuegoHistoria::~JuegoHistoria(){
    #ifdef DEBUG
        cout << "Destructor de JuegoHistoria:"<<this<<endl;
    #endif
}

// JUEGO MOSTRAR GAN 

#include "juego_mostrar_gan.hpp"

JuegoMostrarGanadas::JuegoMostrarGanadas(GameManager * game,JuegoBatalla * parent,int batallasGanadas[_PLAYERS]){
    this->game=game;
    juegoEnCurso=parent;
    for(int i=0;i<_PLAYERS;i++)
        this->batallasGanadas[i]=batallasGanadas[i];
    conteo=0;
    animacion=1;
    fondoJuego=SDL_CreateRGBSurface(SDL_SWSURFACE,W_SCREEN, H_SCREEN, 16,0,0, 0, 255);
    fondoNegro=SDL_CreateRGBSurface(SDL_SWSURFACE,W_SCREEN, H_SCREEN, 16,0,0, 0, 255);
    SDL_FillRect(fondoNegro,0,0);
    dibujar_objeto(SDL_GetVideoSurface(),0,0,fondoJuego);
  	SDL_SetAlpha(fondoNegro, SDL_SRCALPHA|SDL_RLEACCEL,conteo);
  	
  	animaCuadro=new Animacion(game->getImagen(IMG_CUADRO_SCOREBOARD),3,1,"0,0,1,1,2,2",80,H_SCREEN);
  	animaTexto=new Animacion(game->getImagen(IMG_TXT_SCOREBOARD),4,1,"0,0,1,1,2,2,3,3",90,38);

  	int tmp=0;
    for(int i=0;i<_PLAYERS;i++){
        for(int j=0;j<batallasGanadas[i];j++){
            tmp++;
        }
    }
    animaTrofeos=new Animacion*[tmp];
    totalTrofeosCreados=tmp;
    tmp=0;
    for(int i=0;i<_PLAYERS;i++){
        for(int j=0;j<batallasGanadas[i];j++){
            if(tmp+1!=totalTrofeosCreados){
                animaTrofeos[tmp]=new Animacion(game->getImagen(IMG_TROFEO),1,13,"7,7,7,7,7,7,7,7,7,7,8,8,9,9,9,10,10,10,11,11,12,12,12",86+22*j,H_SCREEN+33*i+8,i);
            }else{
                animaTrofeos[tmp]=new Animacion(game->getImagen(IMG_TROFEO),1,13,"0,0,1,1,2,2,2,3,3,4,4,5,5,6,6,7,7,7,7,7,7,7,7,7,7,8,8,9,9,9,10,10,10,11,11,12,12,12",86+22*j,H_SCREEN+35*PLAYER_3+1,i);
                animaTrofeos[tmp]->setCuadroDespues(17);            
            }
            if(i==PLAYER_3)
                animaTrofeos[tmp]->setY(H_SCREEN+35*i+1);
            else if(i==PLAYER_4)
                animaTrofeos[tmp]->setY(H_SCREEN+35*i-4);
            else if(i==PLAYER_5)
                animaTrofeos[tmp]->setY(H_SCREEN+35*i-7);
            else
                animaTrofeos[tmp]->setY(H_SCREEN+35*i+6);
            animaTrofeos[tmp]->setLoop(-1);            
            tmp++;
        }
    }
}
void JuegoMostrarGanadas::procesarEvento(SDL_Event * evento){
    if(animacion==3)
        switch(evento->type){
            case SDL_JOYBUTTONDOWN:if(evento->jbutton.type != SDL_JOYBUTTONDOWN)break;
            case SDL_KEYDOWN:
                salir();
            break;
            
        }

}
void JuegoMostrarGanadas::update(){
    if(animacion==1){
        if((conteo+=3)>=255){
            animacion=2;
            conteo=0;
    	   SDL_SetAlpha(fondoNegro, SDL_SRCALPHA|SDL_RLEACCEL,255);
        }else
        	SDL_SetAlpha(fondoNegro, SDL_SRCALPHA|SDL_RLEACCEL,conteo);
    	
    }else if(animacion==2){
        if(animaCuadro->getY()<=70){
            animacion=3;
            conteo=0;
        }
        animaCuadro->setY(animaCuadro->getY()-9);
        for(int i=0;i<totalTrofeosCreados;i++){
            animaTrofeos[i]->setY(animaTrofeos[i]->getY()-9);
        }
        
    }else if(animacion==3){
        if(++conteo>20){
            animaTrofeos[totalTrofeosCreados-1]->update();
            conteo=20;
        }
    }
    animaCuadro->update();
    animaTexto->update();
    for(int i=0;i<totalTrofeosCreados-1;i++){
        animaTrofeos[i]->update();
    }
}

void JuegoMostrarGanadas::draw(SDL_Surface * screen){
    dibujar_objeto(fondoJuego,0,0,screen);
    dibujar_objeto(fondoNegro,0,0,screen);
    if(animacion!=1)
        animaTexto->draw(screen);
    animaCuadro->draw(screen);
    for(int i=0;i<totalTrofeosCreados-1;i++){
        animaTrofeos[i]->draw(screen);
    }
    if(animacion==3&&conteo>=20)animaTrofeos[totalTrofeosCreados-1]->draw(screen);
    imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN),PLAYER_1*2,screen,64,animaCuadro->getY()+PLAYER_1*35+8,1,10,0);
    imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN),PLAYER_2*2,screen,64,animaCuadro->getY()+PLAYER_2*35+8,1,10,0);
    imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN),PLAYER_3*2,screen,64,animaCuadro->getY()+PLAYER_3*35+6,1,10,0);
    imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN),PLAYER_4*2,screen,64,animaCuadro->getY()+PLAYER_4*35+2,1,10,0);
    imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN),PLAYER_5*2,screen,64,animaCuadro->getY()+PLAYER_5*35,1,10,0);
}

void JuegoMostrarGanadas::salir(){
    game->cambiarInterfaz(juegoEnCurso);
}
JuegoMostrarGanadas::~JuegoMostrarGanadas(){
    #ifdef DEBUG
        cout << "Destructor de JuegoMostrarGanadas"<<endl;
    #endif
    SDL_FreeSurface(fondoJuego);
    SDL_FreeSurface(fondoNegro);
    delete animaCuadro;
    delete animaTexto;
    for(int i=0;i<totalTrofeosCreados;i++){
        delete animaTrofeos[i];
    }
}

// MAIN 

#include "game_manager.hpp"
#include "menu.hpp"
/*
  Name: DestructionBombs
  Copyright: GNU/GPL v3
  Author: Manuel Gonzalez @manuelggz
  Date: 21/04/12 22:14
  Description: Remake del famoso bomberman
  me puedes dar tus comentarios por aqui deathmanuel@gmail.com o en mi blog http://baulprogramas.blogspot.com
  
  si no entiendes algo hazmelo saber con gusto te respondere
  Lineas de codigo a esta fecha:5672
*/

int main(int argc, char *argv[]){
    GameManager * juego=new GameManager();
    if(!juego){
    }
//    juego->cambiarInterfaz(new Menu(juego,Menu::MENU_INICIO));
    juego->cambiarInterfaz(new Menu(juego));
    juego->run();
    delete juego;
    return EXIT_SUCCESS;
}


// MAPA 

#include "mapa.hpp"



Mapa::Mapa(Interfaz * _parent,int coorXVis,int coorYVis,SDL_Surface * grillaTiles){
    #ifdef DEBUG
        cout << "Constructor de Mapa:"<<this<<endl;
    #endif
    this->parent=parent;
    setEjeVisualizacion(coorXVis,coorYVis);
    setImgTiles(grillaTiles);
    datMapa=NULL;
//    mapaCargado=false;
    tileXPuerta=tileYPuerta=-1;
}

bool Mapa::cargarDeArchivoBin(char rutaMapaBin[],char rutaParamText[]){
    tileXPuerta=tileYPuerta=-1; 
    if(!cargarMapaDeArchivoBin(rutaMapaBin,tilesMap))
        return false;

    if(datMapa)delete datMapa;
    datMapa=new DatNivel(rutaParamText);
    if(!datMapa)return false;
    
    bloquesMadera=0;
    for(int i=0;i<MAXMAP;i++)
        if(tilesMap[i]==BLOQUE_MADERA)
            bloquesMadera++;
    char rutaT[50];
    sprintf(rutaT,"data/imagenes/objetos/tile_%d.txt",datMapa->getIdTile()+1);
    leerInfTile(rutaT);

    return true;
}

void Mapa::leerInfTile(char ruta[]){
    char key[50],valor[50];
    ifstream ftile(ruta);
    while(!ftile.eof()){
        ftile >>key;
        if(!strcmp(key,"IMG_FONDO")){
            ftile>>valor;
            if(!strcmp(valor,"FONDO_1"))
                idFondo=IMG_FONDO_PARTI;
            else if(!strcmp(valor,"FONDO_2"))
                idFondo=IMG_FONDO_EDIFICIOS;
            else if(!strcmp(valor,"FONDO_3"))
                idFondo=IMG_FONDO_METAL;
        }else if(!strcmp(key,"EJES")){
            ftile>>coorXPredVisualizacion>>coorYPredVisualizacion;
        }else if(!strcmp(key,"Y_TABLERO")){
            ftile>>yTablero;
        }else{
            ftile>>valor;
        }
    }
    ftile.close();
    
}

bool Mapa::cargarMapaDeArchivoBin(char rutaMapaBin[],char * buffer){

    ifstream fileMapa(rutaMapaBin,ios::in|ios::binary);
    if(!fileMapa){
        cerr << "WARNING-Error leyendo un archivo: -- Ruta: "<<rutaMapaBin<<endl;
        return false;
    }
    fileMapa.read(reinterpret_cast<char *> (buffer),MAXMAP);
    fileMapa.close();
    
    return true;
}

void Mapa::draw(SDL_Surface * screen,SDL_Surface * tiles,char * mapa,int coorX,int coorY,int idTile)
{
    int indice;
    SDL_Rect dest={0,0,16,16};

    for(int i=0;i<FILAS;i++)
        for(int j=0;j<COLUMNAS;j++){
            // calculo de la posición del tile
            dest.x = j * SIZE_TILE+coorX;
            dest.y= i * SIZE_TILE+coorY;

            indice=mapa[i*COLUMNAS+j];
            if(indice==BLOQUE_ITEM)
                indice=BLOQUE_MADERA;
            else if(indice==BLOQUE_ENEMIGO)
                indice=BLOQUE_PISO;
            if(dest.x+SIZE_TILE>=0&&dest.y+SIZE_TILE>=0&&dest.x<W_SCREEN&&dest.y<H_SCREEN)
        	imprimir_desde_grilla (tiles,idTile*4+ indice,screen,dest.x,dest.y,4,4,0);
        }
}

int Mapa::setItems(){

    int itemsPuestos=0;
    int random_x=0,random_y=0;

    while(itemsPuestos < datMapa->getNumItems()){
            if(bloquesMadera>0){
                do{
                    random_x=rand()%COLUMNAS;
                    random_y=rand()%FILAS;
                }while(tilesMap[random_y*COLUMNAS+random_x]!=BLOQUE_MADERA);
    
                tilesMap[random_y*COLUMNAS+random_x]=BLOQUE_ITEM;
                bloquesMadera--;
                if(tileXPuerta==-1||tileYPuerta==-1){ /*Si aun no se ha establecido la puerta nota:la segunda condicional nunca se evaluara*/
                    tileXPuerta=random_x;
                    tileYPuerta=random_y;
                }
                itemsPuestos++;
            }
            else{
                break;
            }
    }
    datMapa->setNumItems(itemsPuestos);
    return itemsPuestos;

}
int Mapa::setEnemigos(){
    int indice,colocados=0;
    for(int i=0;i<FILAS;i++)
        for(int j=0;j<COLUMNAS;j++){
            indice=tilesMap[i*COLUMNAS+j];
            if(indice==BLOQUE_ENEMIGO){
                  tilesMap[i*COLUMNAS+j]=BLOQUE_PISO;
                  colocados++;
//                  parent->addSprite(GLOBO,2,j * W_H_BLOQUE+EJE_X,i * W_H_BLOQUE+EJE_Y);
            }
        }
    return colocados;
}

bool Mapa::isBloqueRompible(int x,int y){

    int tipoBloque=getTipoBloque(x,y);
    if(tipoBloque!=-1)
        return tipoBloque==BLOQUE_ITEM||tipoBloque==BLOQUE_MADERA;
    
    cout << "Warning: Acceso invalido al mapa,X:"<<x <<"Y:"<<y<<endl;
    return false;
}

bool Mapa::romperBloque(int x,int y){
    if(isBloqueRompible(x,y)){
        if((y==getEjeYVisualizacion())||\
           (y!=getEjeYVisualizacion()&&getTipoBloque(x,y-16)!=BLOQUE_PISO&&getTipoBloque(x,y-16)!=BLOQUE_PISO_SOMBRA))
            tilesMap[getIndiceMapa(x,y)]=BLOQUE_PISO_SOMBRA;
        else
            tilesMap[getIndiceMapa(x,y)]=BLOQUE_PISO;
        if(y+16!=getEjeYVisualizacion()+getAltoMapa()&&getTipoBloque(x,y+16)==BLOQUE_PISO_SOMBRA)
            tilesMap[getIndiceMapa(x,y+16)]=BLOQUE_PISO;
        return true;
    }
    return false;
}


bool Mapa::colision(SDL_Rect * rect, int * num_colisiones,bool solo_bloques_duros)
{//solo detecta la colision en las esquinas del rect

    int ret=0;
    int indice;
    int left=rect->x-coorXVisualizacion,
        top=rect->y-coorYVisualizacion;
    
    for(int i=0;i<4;i++){
        indice=(((top+rect->h*(i>=2))/SIZE_TILE)*COLUMNAS+((left + (rect->w*(i!=0&&i!=3)))/SIZE_TILE));
        
        if(!solo_bloques_duros){
            if(tilesMap[indice]!=BLOQUE_PISO&&tilesMap[indice]!=BLOQUE_PISO_SOMBRA&&tilesMap[indice]!=BLOQUE_ENEMIGO){
                (*num_colisiones)++;
                ret=i+1;
            }
        }else if(solo_bloques_duros&&tilesMap[indice]==BLOQUE_METAL){
            (*num_colisiones)++;
            ret=i+1;
        }
    }

    return ret;

}
Mapa::~Mapa(){
    #ifdef DEBUG
        cout << "Destructor de Mapa:"<<this<<endl;
    #endif
    delete datMapa;
}


SDL_Surface * Mapa::getPreviewTerreno(char rutaMapa[],DatNivel * params,SDL_Surface * img_tile,SDL_Surface * imgs_players[],int x,int y){

    SDL_Surface * preview=SDL_GetVideoSurface(),*imagen_redimensionada;
    char ruta[50],mapa[MAXMAP + 1],variable[50];
    int i;
    SDL_Rect rect_destino;

//    sprintf(ruta,"data/niveles/batalla/%d.map",id + 1);
    cargarMapaDeArchivoBin(rutaMapa,mapa);

    SDL_FillRect(preview,0,0);
    if(params)
        draw(preview,img_tile,mapa,x,y,params->getIdTile());
    else
        draw(preview,img_tile,mapa,x,y,0);

    if(params){
//        sprintf(ruta,"data/niveles/batalla/%d.ini",id + 1);

        for(i=0;i<_PLAYERS;i++){
            rect_destino.x=params->getX(i);
            rect_destino.y=params->getY(i);
            imprimir_desde_grilla(imgs_players[i],6,preview,rect_destino.x,rect_destino.y,1,12,0);
        }

    }


    imagen_redimensionada=(SDL_Surface * )zoomSurface(preview, 0.3515, 0.4979, 1);
    SDL_FreeSurface(preview);
    return imagen_redimensionada;
}
/*
TipoItem Nivel::getTipoNuevoItem(InterfazJuego inter){

    static TipoItem tipos[5]={ITEM_ALCANCE,ITEM_VIDA,ITEM_BOMBA,ITEM_PARED,ITEM_ALEATORIO};
    static int salido_anterior=-1;

    TipoItem indice=ITEM_ALCANCE;

    if(dat_nivel->getNumItems()>=1){
        switch(inter){
            case TIPO_NORMAL:
                 do
                    indice=tipos[rand() % 5 ];
                 while(indice==salido_anterior);
                salido_anterior=indice;
                break;
            case TIPO_BATALLA:
                do{
                    indice=tipos[rand() % 5 ];
                }while(indice==ITEM_PUERTA||indice==ITEM_VIDA||indice==ITEM_ALEATORIO);//items no permitidos en modo batalla

                break;
            }

       dat_nivel->setNumItems(dat_nivel->getNumItems()-1); 
    }

    return indice;
}*/

/*
SDL_Surface * Mapa::getPreviewTerreno(int idTerreno,Interfaz * game){
    SDL_Surface * preview;
    SDL_Surface * img_players[5]={game->getImagen(IMG_PLAYER_1),
                              game->getImagen(IMG_PLAYER_2),
                              game->getImagen(IMG_PLAYER_3),
                              game->getImagen(IMG_PLAYER_4),
                              game->getImagen(IMG_PLAYER_5)};
                              
    sprintf(ruta,"data/niveles/batalla/%d.txt",idTerreno+ 1);
    data2=new DatNivel(ruta);
    sprintf(ruta,"data/niveles/batalla/%d.map",idTerreno+ 1);
    preview=Mapa::getPreviewTerreno(ruta,data2,game->getImagen((CodeImagen)(IMG_TILE_1+data2->getIdTile())),img_players,EJE_X,EJE_Y);
//        cout << "Creada: "<<previews_niveles[i]<<endl;
    delete data2;
}
*/
/*
void Nivel::cargarFiles(int num_nivel,InterfazJuego inter)
{
    char ruta1[50],ruta2[50];
    
    switch(inter){
        case TIPO_NORMAL:
            sprintf(ruta1,"data/niveles/historia/nivel_%d.map",num_nivel);
            sprintf(ruta2,"data/niveles/historia/%d.txt",num_nivel);
            break;
        case TIPO_BATALLA:
            sprintf(ruta1,"data/niveles/batalla/%d.map",num_nivel);
            sprintf(ruta2,"data/niveles/batalla/%d.txt",num_nivel);
            break;
        }
    cargarFileNivel(mapa,ruta1);
    
    if(dat_nivel)delete dat_nivel;
    dat_nivel=new DatNivel(ruta2);
}
*/


// MENU 

#include "menu.hpp"

Menu::Menu(GameManager * game):
    botonGuardar(game->getImagen(IMG_BOTON_GUARDAR),this,114,205),mapa(this){
    #ifdef DEBUG
    cout << "Constructor de Menu:"<<this<<endl;
    #endif
    this->game=game;

    
    // ventana 1
    strcpy(texto[VENTANA_1][MENU_NUEVO_JUEGO],"NUEVO JUEGO");
    strcpy(texto[VENTANA_1][MENU_EDITOR],"EDITOR");
    strcpy(texto[VENTANA_1][MENU_CONFIGURACION],"CONFIGURAR");
    strcpy(texto[VENTANA_1][MENU_CREDITOS],"CREDITOS");
    strcpy(texto[VENTANA_1][MENU_SALIR],"SALIR");

    //ventana 2
    strcpy(texto[VENTANA_2][MENU_MODO_HISTORIA],"HISTORIA");
    strcpy(texto[VENTANA_2][MENU_MODO_MULTIJUGADOR],"MULTIPLAYER");
    strcpy(texto[VENTANA_2][MENU_MODO_CONEXION],"CONEXIÓN");
    strcpy(texto[VENTANA_2][MENU_REGRESAR],"REGRESAR");


    /*RECTS PARA IMPRIMIR LOS MENSAGES DE LA VENTANA*/
    //ventana 1
    rectsImpresion[VENTANA_1][MENU_NUEVO_JUEGO].x=38;
    rectsImpresion[VENTANA_1][MENU_NUEVO_JUEGO].y=77;

    rectsImpresion[VENTANA_1][MENU_EDITOR].x=102;
    rectsImpresion[VENTANA_1][MENU_EDITOR].y=115;

    rectsImpresion[VENTANA_1][MENU_CONFIGURACION].x=49;
    rectsImpresion[VENTANA_1][MENU_CONFIGURACION].y=145;

    rectsImpresion[VENTANA_1][MENU_CREDITOS].x=102;
    rectsImpresion[VENTANA_1][MENU_CREDITOS].y=175;

    rectsImpresion[VENTANA_1][MENU_CREDITOS].x=64;
    rectsImpresion[VENTANA_1][MENU_CREDITOS].y=175;

    rectsImpresion[VENTANA_1][MENU_SALIR].x=91;
    rectsImpresion[VENTANA_1][MENU_SALIR].y=200;

    //ventana 2
    rectsImpresion[VENTANA_2][MENU_MODO_HISTORIA].x=66;
    rectsImpresion[VENTANA_2][MENU_MODO_HISTORIA].y=77;

    rectsImpresion[VENTANA_2][MENU_MODO_MULTIJUGADOR].x=42;
    rectsImpresion[VENTANA_2][MENU_MODO_MULTIJUGADOR].y=115;

    rectsImpresion[VENTANA_2][MENU_MODO_CONEXION].x=66;
    rectsImpresion[VENTANA_2][MENU_MODO_CONEXION].y=153;

    rectsImpresion[VENTANA_2][MENU_REGRESAR].x=69;
    rectsImpresion[VENTANA_2][MENU_REGRESAR].y=195;

    //ventana 3 (MULTIJUGADOR)
    sprites=new Group(this);

    btnSubirTiempo=new Boton<Menu>(game->getImagen(IMG_BOTON_FLECHA_PEQUE_DERECHA),this,194,8);
    btnSubirVictorias=new Boton<Menu>(game->getImagen(IMG_BOTON_FLECHA_PEQUE_DERECHA),this,295,8);
    btnCambiarMapa=new Boton<Menu>(game->getImagen(IMG_BOTON_CAMBIAR_MAPA),this,160,225);
    btnJugar=new Boton<Menu>(game->getImagen(IMG_BOTON_JUGAR_2),this,240,225);
    
    btnSubirTiempo->setId(MENU_BOTON_SUBIR_TIEMPO);
    btnSubirVictorias->setId(MENU_BOTON_SUBIR_VICTORIAS);
    btnCambiarMapa->setId(MENU_BOTON_CAMBIAR_MAPA);
    btnJugar->setId(MENU_BOTON_JUGAR);
    
    btnSubirTiempo->bindAccion(&Menu::clickControl);
    btnSubirVictorias->bindAccion(&Menu::clickControl);
    btnCambiarMapa->bindAccion(&Menu::clickControl);
    btnJugar->bindAccion(&Menu::clickControl);    
    btnJugar->setVisible(false);
    
    animaPlayer[0]=new Animacion(game->getImagen(IMG_PLAYER_1),1,12,"6,6,7,7,8,8",X_INIT_PLAYER_1,Y_INIT_PLAYER_1,0);
    animaPlayer[1]=new Animacion(game->getImagen(IMG_PLAYER_2),1,12,"6,6,7,7,8,8",X_INIT_PLAYER_2,Y_INIT_PLAYER_2,1);
    animaPlayer[2]=new Animacion(game->getImagen(IMG_PLAYER_3),1,12,"6,6,7,7,8,8",X_INIT_PLAYER_3,Y_INIT_PLAYER_3,2);
    animaPlayer[3]=new Animacion(game->getImagen(IMG_PLAYER_4),1,12,"6,6,7,7,8,8",X_INIT_PLAYER_4,Y_INIT_PLAYER_4-20,3);
    animaPlayer[4]=new Animacion(game->getImagen(IMG_PLAYER_5),1,12,"6,6,7,7,8,8",X_INIT_PLAYER_5,Y_INIT_PLAYER_5,4);

    for(int i=0;i<_PLAYERS;i++){
        animaPresiona[i]=new Animacion(game->getImagen(IMG_TXT_PRESIONA),2,1,"0,0,1,1",animaPlayer[i]->getX()-9,animaPlayer[i]->getY()+20,i);
        animaActivado[i]=new Animacion(game->getImagen(IMG_TXT_ACTIVADO),2,1,"0,0,0,1,1,1",animaPlayer[i]->getX()-9,animaPlayer[i]->getY()+20,i);
        animaPlayer[i]->setLoop(-1);
        animaActivado[i]->setLoop(-1);
        animaPresiona[i]->setLoop(-1);
        sprites->add(animaPresiona[i]);
    }
    
    /*ventana 4*/

    //linea de tecla arriba
    rectConfiguracion[MENU_TECLA_ARRIBA][MENU_BOTON_CAMBIAR].x=190;
    rectConfiguracion[MENU_TECLA_ARRIBA][MENU_BOTON_CAMBIAR].y=68;
    rectConfiguracion[MENU_TECLA_ARRIBA][MENU_BOTON_CAMBIAR].w=81;
    rectConfiguracion[MENU_TECLA_ARRIBA][MENU_BOTON_CAMBIAR].h=18;


    rectConfiguracion[MENU_TECLA_ARRIBA][MENU_CUADRO_MOSTRAR].x=93;
    rectConfiguracion[MENU_TECLA_ARRIBA][MENU_CUADRO_MOSTRAR].y=70;

    rectConfiguracion[MENU_TECLA_ARRIBA][MENU_TEXTO_MOSTRAR].x=27;
    rectConfiguracion[MENU_TECLA_ARRIBA][MENU_TEXTO_MOSTRAR].y=69;

    //linea de tecla abajo
    rectConfiguracion[MENU_TECLA_ABAJO][MENU_BOTON_CAMBIAR].x=190;
    rectConfiguracion[MENU_TECLA_ABAJO][MENU_BOTON_CAMBIAR].y=91;
    rectConfiguracion[MENU_TECLA_ABAJO][MENU_BOTON_CAMBIAR].w=81;
    rectConfiguracion[MENU_TECLA_ABAJO][MENU_BOTON_CAMBIAR].h=18;

    rectConfiguracion[MENU_TECLA_ABAJO][MENU_CUADRO_MOSTRAR].x=93;
    rectConfiguracion[MENU_TECLA_ABAJO][MENU_CUADRO_MOSTRAR].y=93;

    rectConfiguracion[MENU_TECLA_ABAJO][MENU_TEXTO_MOSTRAR].x=27;
    rectConfiguracion[MENU_TECLA_ABAJO][MENU_TEXTO_MOSTRAR].y=93;

    //linea de tecla izquierda
    rectConfiguracion[MENU_TECLA_IZQUIERDA][MENU_BOTON_CAMBIAR].x=190;
    rectConfiguracion[MENU_TECLA_IZQUIERDA][MENU_BOTON_CAMBIAR].y=112;
    rectConfiguracion[MENU_TECLA_IZQUIERDA][MENU_BOTON_CAMBIAR].w=81;
    rectConfiguracion[MENU_TECLA_IZQUIERDA][MENU_BOTON_CAMBIAR].h=18;

    rectConfiguracion[MENU_TECLA_IZQUIERDA][MENU_CUADRO_MOSTRAR].x=93;
    rectConfiguracion[MENU_TECLA_IZQUIERDA][MENU_CUADRO_MOSTRAR].y=112;

    rectConfiguracion[MENU_TECLA_IZQUIERDA][MENU_TEXTO_MOSTRAR].x=27;
    rectConfiguracion[MENU_TECLA_IZQUIERDA][MENU_TEXTO_MOSTRAR].y=112;

    //linea de tecla derecha
    rectConfiguracion[MENU_TECLA_DERECHA][MENU_BOTON_CAMBIAR].x=190;
    rectConfiguracion[MENU_TECLA_DERECHA][MENU_BOTON_CAMBIAR].y=133;
    rectConfiguracion[MENU_TECLA_DERECHA][MENU_BOTON_CAMBIAR].w=81;
    rectConfiguracion[MENU_TECLA_DERECHA][MENU_BOTON_CAMBIAR].h=18;

    rectConfiguracion[MENU_TECLA_DERECHA][MENU_CUADRO_MOSTRAR].x=93;
    rectConfiguracion[MENU_TECLA_DERECHA][MENU_CUADRO_MOSTRAR].y=133;

    rectConfiguracion[MENU_TECLA_DERECHA][MENU_TEXTO_MOSTRAR].x=27;
    rectConfiguracion[MENU_TECLA_DERECHA][MENU_TEXTO_MOSTRAR].y=133;


    //linea de tecla accion
    rectConfiguracion[MENU_TECLA_ACCION][MENU_BOTON_CAMBIAR].x=190;
    rectConfiguracion[MENU_TECLA_ACCION][MENU_BOTON_CAMBIAR].y=153;
    rectConfiguracion[MENU_TECLA_ACCION][MENU_BOTON_CAMBIAR].w=81;
    rectConfiguracion[MENU_TECLA_ACCION][MENU_BOTON_CAMBIAR].h=18;

    rectConfiguracion[MENU_TECLA_ACCION][MENU_CUADRO_MOSTRAR].x=93;
    rectConfiguracion[MENU_TECLA_ACCION][MENU_CUADRO_MOSTRAR].y=153;

    rectConfiguracion[MENU_TECLA_ACCION][MENU_TEXTO_MOSTRAR].x=27;
    rectConfiguracion[MENU_TECLA_ACCION][MENU_TEXTO_MOSTRAR].y=153;

    //linea de tecla start
    rectConfiguracion[MENU_TECLA_START][MENU_BOTON_CAMBIAR].x=190;
    rectConfiguracion[MENU_TECLA_START][MENU_BOTON_CAMBIAR].y=173;
    rectConfiguracion[MENU_TECLA_START][MENU_BOTON_CAMBIAR].w=81;
    rectConfiguracion[MENU_TECLA_START][MENU_BOTON_CAMBIAR].h=18;

    rectConfiguracion[MENU_TECLA_START][MENU_CUADRO_MOSTRAR].x=93;
    rectConfiguracion[MENU_TECLA_START][MENU_CUADRO_MOSTRAR].y=173;

    rectConfiguracion[MENU_TECLA_START][MENU_TEXTO_MOSTRAR].x=27;
    rectConfiguracion[MENU_TECLA_START][MENU_TEXTO_MOSTRAR].y=173;


    //botones para cambiar de player

    botonPlayer[PLAYER_1]=new Boton<Menu>(game->getImagen(IMG_BOTON_PLAYER_1),this,3,228);
    botonPlayer[PLAYER_2]=new Boton<Menu>(game->getImagen(IMG_BOTON_PLAYER_2),this,64,228);
    botonPlayer[PLAYER_3]=new Boton<Menu>(game->getImagen(IMG_BOTON_PLAYER_3),this,125,228);
    botonPlayer[PLAYER_4]=new Boton<Menu>(game->getImagen(IMG_BOTON_PLAYER_4),this,189,228);
    botonPlayer[PLAYER_5]=new Boton<Menu>(game->getImagen(IMG_BOTON_PLAYER_5),this,252,228);

    botonGuardar.bindAccion(&Menu::guardarTeclas);
    for(int i=0;i<_PLAYERS;i++)
        botonPlayer[i]->bindAccion(&Menu::cambiarPlayer);

    rect_destino_cara.x=126;
    rect_destino_cara.y=50;


    botones_cambiar[MENU_TECLA_ARRIBA]=BOTON_NORMAL;
    botones_cambiar[MENU_TECLA_ABAJO]=BOTON_NORMAL;
    botones_cambiar[MENU_TECLA_IZQUIERDA]=BOTON_NORMAL;
    botones_cambiar[MENU_TECLA_DERECHA]=BOTON_NORMAL;
    botones_cambiar[MENU_TECLA_ACCION]=BOTON_NORMAL;
    botones_cambiar[MENU_TECLA_START]=BOTON_NORMAL;


//    estado_boton_guardar=BOTON_NORMAL;
    maxTerrenoBatalla=buscar_dato(RUTA_CONFIG_BASE,"MaxTerreno");
    player_configurando_teclas=PLAYER_NONE;
    previewTerreno=NULL;
    mapa.setImgTiles(game->getImagen(IMG_TILES));
    limpiar();
    updatePreview();
    setSelected(0);
    game->playSonido(SND_MENU);
    dataNivel=NULL;;
//    cambiarVentana(VENTANA_1);

    fondoVentanaAnterior=SDL_CreateRGBSurface(SDL_SWSURFACE,W_SCREEN, H_SCREEN, 24,0,0, 0, 255);
    fondoVentanaSiguiente=SDL_CreateRGBSurface(SDL_SWSURFACE,W_SCREEN, H_SCREEN, 24,0,0, 0, 255);
    fondoNegro=SDL_CreateRGBSurface(SDL_SWSURFACE,W_SCREEN, H_SCREEN, 24,0,0, 0, 0);
    animacion=false;
    setDesvanecimiento(-1,VENTANA_1);

}
void Menu::setDesvanecimiento(int last,int nueva){
  	ventanaAnterior=last;
  	ventanaSiguiente=nueva;
    nivelAlpha=0;
    nAnimacion=1;
    if(last==-1){
        nAnimacion=2;
        nivelAlpha=255;
    }else
        dibujar_objeto(SDL_GetVideoSurface(),0,0,fondoVentanaAnterior);
    if(ventanaSiguiente!=-1){
        cambiarVentana(ventanaSiguiente);
        draw(fondoVentanaSiguiente);
    }
    SDL_SetAlpha(fondoNegro, SDL_SRCALPHA|SDL_RLEACCEL,nivelAlpha);
    animacion=true;
}

void Menu::setSelected(int nuevo){
    if(nuevo<((ventana==VENTANA_1)?5:4)&&nuevo>=0)
        selected=nuevo;
    game->play(SFX_TONO_ACUATICO);
}

void Menu::cambiarPlayer(){
    for(int i=0;i<_PLAYERS;i++){
        if(i!=player_configurando_teclas&&botonPlayer[i]->estaPresionado()){
            cambiarPlayerConfi((IdPlayer)i);          
            return;
        }
    }
}
void Menu::cambiarPlayerConfi(IdPlayer id){
     //cambia el estado de las variables asociadas al cambio de player al que se le configuran las teclas
      if(player_configurando_teclas!=PLAYER_NONE){
          botonPlayer[player_configurando_teclas]->setEstado(Boton<Menu>::NORMAL);
          botonPlayer[player_configurando_teclas]->setEnable(true);  
      }      
      player_configurando_teclas=id;// guarda el id del nuevo player
      id_espera_tecla=TECLA_NULA; //variable que dice si se espera la pulsacion de una tecla[del joy] para un boton del juego
      botonPlayer[id]->setEstado(Boton<Menu>::PRESIONADO);
      botonPlayer[id]->setEnable(false);
      cargarTeclas();//carga las teclas del File 
}

void Menu::cargarTeclas(){
     //carga las teclas de una player
    char str_tmp[40];
    sprintf(str_tmp,"data/configuracion/teclado_%d.dat",player_configurando_teclas+1);
    control_edit.cargar(str_tmp);

}
void Menu::guardarTeclas(){
    char tmp[40];
    sprintf(tmp,"data/configuracion/teclado_%d.dat",player_configurando_teclas+1);
    control_edit.guardar(tmp);
}

void Menu::limpiar(){
    cambiarPlayerConfi(PLAYER_1);
    for(int i=0;i<_PLAYERS;i++)
          player_batalla[i]=false;
     terrenoActual=0;
     minutosEscogidos=1;
     victoriasEscogidas=1;
}
void Menu::clickControl(Boton<Menu> * control_click){
    selected=control_click->getId();
    clickSelected();
}


void Menu::cambiarVentana(int nueva_ventana){
    ventana=nueva_ventana;
    selected=0;
    SDL_ShowCursor(SDL_DISABLE);//ocultamos el cursor
    
    if(ventana==VENTANA_3){//si es la de multijugador
        SDL_ShowCursor(SDL_ENABLE);
    }
    else if(ventana==VENTANA_4){ //si es la de configuracion
        cargarTeclas();
        id_espera_tecla=TECLA_NULA;
        SDL_ShowCursor(SDL_ENABLE);
//        game->cambiarInterfaz(new EfectoDesvanecimiento(game,this,this));
    }
}


void Menu::cambiarTerrenoBatalla(int aum){
    terrenoActual+=aum;
    if(terrenoActual==maxTerrenoBatalla)//si es el ultimo
        terrenoActual=0;//ponemos el primero
    else if(terrenoActual<0)
        terrenoActual=maxTerrenoBatalla-1;//ponemos el ultimo

}

void Menu::clickSelected(){
    int tmp;

    switch(ventana){
        case VENTANA_1:
            switch(selected){
                case MENU_NUEVO_JUEGO:
//                    cambiarVentana(VENTANA_2);
                    setDesvanecimiento(VENTANA_1,VENTANA_2);
                    break;
                case MENU_EDITOR:
                     game->cambiarInterfaz(new Editor(game));
                    break;
                case MENU_CONFIGURACION:
//                    cambiarVentana(VENTANA_4);
                    setDesvanecimiento(VENTANA_1,VENTANA_4);
                    break;
                case MENU_CREDITOS:
//                    cambiarVentana(VENTANA_CREDITOS);
                    setDesvanecimiento(VENTANA_1,VENTANA_CREDITOS);
                    break;
                case MENU_SALIR:
                    setDesvanecimiento(VENTANA_1,-1);
//                    game->quit();
                    break;
                }
            break;
        case VENTANA_2:
            switch(selected){
                case MENU_MODO_HISTORIA:
                    game->cambiarInterfaz(new JuegoHistoria(game));
                    break;
                case MENU_MODO_MULTIJUGADOR:
                    setDesvanecimiento(VENTANA_2,VENTANA_3);
//                    cambiarVentana(VENTANA_3);
                    break;
                case MENU_MODO_CONEXION:
                    break;
                case MENU_REGRESAR:
                    setDesvanecimiento(VENTANA_2,VENTANA_1);
//                    cambiarVentana(VENTANA_1);
                    break;
                }
            break;
        case VENTANA_3:
                switch(selected){
                    case PLAYER_1:case PLAYER_2:case PLAYER_3:case PLAYER_4:case PLAYER_5:
                          player_batalla[selected]=!player_batalla[selected];
                          if(player_batalla[selected]){
                                sprites->add(animaPlayer[selected]);
                                sprites->add(animaActivado[selected]);
                                sprites->erase(animaPresiona[selected]);
                          }else{
                                sprites->erase(animaPlayer[selected]);
                                sprites->erase(animaActivado[selected]);
                                sprites->add(animaPresiona[selected]);
                          }
                          btnJugar->setVisible(player_batalla[PLAYER_1]+ player_batalla[PLAYER_2]+ player_batalla[PLAYER_3] + player_batalla[PLAYER_4] + player_batalla[PLAYER_5]>1);
                          game->play(SFX_TONO_SECO);
                      break;
                    case MENU_BOTON_SUBIR_TIEMPO:
                        if(++minutosEscogidos>5)minutosEscogidos=1;
                        break;
                    case MENU_BOTON_SUBIR_VICTORIAS:
                        if(++victoriasEscogidas>8)victoriasEscogidas=1;
                        break;
                    case MENU_BOTON_CAMBIAR_MAPA:
                        if(++terrenoActual==maxTerrenoBatalla)terrenoActual=0;
                        updatePreview();
                        game->play(SFX_TONO_ACUATICO);
                        break;
                    case MENU_BOTON_JUGAR:
                        int total_players=player_batalla[PLAYER_1]+ player_batalla[PLAYER_2]+ player_batalla[PLAYER_3] + player_batalla[PLAYER_4] + player_batalla[PLAYER_5];
                        if(total_players>=2){
                            game->cambiarInterfaz(new JuegoBatalla(game,terrenoActual,player_batalla,minutosEscogidos,victoriasEscogidas)); //iniciamos en modo batalla, le pasamos el array con los players seleccionados por el usuario
                            game->play(SFX_EXPLOSION);
                        }
                        break;
                }
            break;
            }
}
void Menu::updatePreview(){
    static char ruta1[50],ruta2[50];
    sprintf(ruta1,"data/niveles/batalla/%d.map",terrenoActual+1);
    sprintf(ruta2,"data/niveles/batalla/%d.txt",terrenoActual+1);
    mapa.cargarDeArchivoBin(ruta1,ruta2);
    mapa.setEjeVisualizacion(mapa.getEjeX(),mapa.getEjeY());

}
void Menu::update(){
    if(!animacion){
        switch(ventana){
                case VENTANA_1:case VENTANA_2:
                    break;
                case VENTANA_3://multijugador
                    sprites->update();
                    break;
                case VENTANA_4://configuracion
                    break;
                case VENTANA_CREDITOS://configuracion
                    break;
            }
    }else{
        if(nAnimacion==1&&(nivelAlpha+=6)>=255){
            nAnimacion=2;
            nivelAlpha=255;
            if(ventanaSiguiente==-1)game->quit();
        }else if(nAnimacion==2&&(nivelAlpha-=6)<=0){
            animacion=false;
        }
        SDL_SetAlpha(fondoNegro, SDL_SRCALPHA|SDL_RLEACCEL,nivelAlpha);
    }
}


void Menu::procesarEvento(SDL_Event * evento){
    if(!animacion){
        switch(ventana){
            case VENTANA_1:case VENTANA_2:
                if(evento->type==SDL_KEYDOWN){
                        switch(evento->key.keysym.sym){
                            case SDLK_ESCAPE:
                                 if(ventana==VENTANA_2){
                                    setDesvanecimiento(-1,VENTANA_1);
                                 }else{
                                    setDesvanecimiento(ventana,-1);
                                 }
                                break;
                            case SDLK_DOWN:case SDLK_UP:
                                setSelected(selected + (int)pow(-1,evento->key.keysym.sym-272));
                                break;
                            case SDLK_RETURN:
                                clickSelected();
                                break;
                            default:
                                break;
                            }
                }else if(evento->type == SDL_JOYAXISMOTION&&evento->jaxis.type == SDL_JOYAXISMOTION){
                        if(evento->jaxis.axis != 0){
                             if(evento->jaxis.value > 10){
                               setSelected(selected + 1);
                            }else if(evento->jaxis.value < -10){
                               setSelected(selected - 1);
                            }
                        }
                }else if(evento->type == SDL_JOYBUTTONDOWN){
                     if(evento->jbutton.type == SDL_JOYBUTTONDOWN)
                        if(evento->jbutton.button + 1==3)
                                clickSelected();
    
                }
    
            break;
            case VENTANA_3://Multijugador
                if(evento->type==SDL_KEYDOWN){
                    switch(evento->key.keysym.sym){
                        case SDLK_ESCAPE:
                            setDesvanecimiento(-1,VENTANA_2);
                            limpiar();
                            break;
                        case SDLK_RETURN:
    						selected=MENU_BOTON_JUGAR;
    						clickSelected();
                            break;
                        case SDLK_KP1:case SDLK_KP2:case SDLK_KP3:case SDLK_KP4:case SDLK_KP5:
    						selected=(evento->key.keysym.sym-SDLK_KP1); //seleccionamos el player con una formula mate. SDK_1:49 y SDLK_5:53
    						clickSelected();
    						break;
                        case SDLK_1:case SDLK_2:case SDLK_3:case SDLK_4:case SDLK_5:
    						selected=(evento->key.keysym.sym-SDLK_1); //seleccionamos el player con una formula mate. SDK_1:49 y SDLK_5:53
    						clickSelected();
                            break;
                        case SDLK_LEFT:case SDLK_RIGHT:
    					    selected=MENU_BOTON_CAMBIAR_MAPA;
    						clickSelected();
                            break;
                        default:
                            break;
                    }
                }
                btnSubirTiempo->procesarEvento(evento);
                btnSubirVictorias->procesarEvento(evento);
                btnCambiarMapa->procesarEvento(evento);
                btnJugar->procesarEvento(evento);
                break;
                
            case VENTANA_4:
                        if(evento->type == SDL_JOYBUTTONDOWN && id_espera_tecla >-1){
                            if(evento->jbutton.type == SDL_JOYBUTTONDOWN){
                                    control_edit.setKey(id_espera_tecla,(SDLKey)evento->jbutton.button);
                                    control_edit.setIsBotonJoystick(id_espera_tecla,true);;
                                    control_edit.setIsDireccionJoystick((int)id_espera_tecla,false);
                                    control_edit.setName(id_espera_tecla,SDL_JoystickName(evento->jbutton.which));
                                    id_espera_tecla=TECLA_NULA;//dejamos de esperar a que el usuario presione una tecla
                                }
    
                        }else if(evento->type == SDL_JOYAXISMOTION && id_espera_tecla >= 0){
                                bool modificado=false;
                                if(evento->jaxis.axis == 0) {
                                     if(evento->jaxis.value > 0){
                                            control_edit.setKey(id_espera_tecla,SDLK_RIGHT);
                                            modificado=1;
                                        }
    
                                     else if(evento->jaxis.value < 0){
                                            control_edit.setKey(id_espera_tecla,SDLK_LEFT);
                                            modificado=1;
                                        }
                                } else {
    
                                     if(evento->jaxis.value > 0){
                                            control_edit.setKey(id_espera_tecla,SDLK_DOWN);
                                       modificado=1;
                                    }else if(evento->jaxis.value < 0){
                                            control_edit.setKey(id_espera_tecla,SDLK_UP);
                                            modificado=1;
                                    }
                                }
    
                                if(modificado){
                                    control_edit.setIsBotonJoystick(id_espera_tecla,false);;
                                    control_edit.setIsDireccionJoystick((int)id_espera_tecla,true);
                                    control_edit.setName(id_espera_tecla,SDL_JoystickName(evento->jaxis.which));
                                    id_espera_tecla=TECLA_NULA;
                                }
                        }else if(evento->type==SDL_KEYDOWN){
                                switch(evento->key.keysym.sym){
                                    case SDLK_RIGHT:
                                        if(player_configurando_teclas==PLAYER_5)break;
                                    case SDLK_LEFT:
                                        if(evento->key.keysym.sym==SDLK_LEFT&&player_configurando_teclas==PLAYER_1)break;
                                        if(id_espera_tecla==TECLA_NULA)
                                            cambiarPlayerConfi((IdPlayer)(player_configurando_teclas + (int)pow(-1,evento->key.keysym.sym-271)));//el pow es para tener 1 o -1 dependiendo de la tecla, notar que si es SDLK_LEFT sera -1
                                        break;
    
                                    case SDLK_ESCAPE:
                                        if(id_espera_tecla !=TECLA_NULA){//si se esperaba la pulsacion de una tecla
                                            id_espera_tecla=TECLA_NULA;
                                        }else{
                                            setDesvanecimiento(-1,VENTANA_1);
                                            limpiar();
                                        }
                                        break;
                                    default:
                                        break;
                                }
    
                                if(id_espera_tecla !=TECLA_NULA){
                                    control_edit.setKey(id_espera_tecla,evento->key.keysym.sym);
                                    control_edit.setIsBotonJoystick(id_espera_tecla,false);;
                                    control_edit.setIsDireccionJoystick((int)id_espera_tecla,false);
                                    id_espera_tecla=TECLA_NULA;
                                }
                        }
                        else if(evento->type == SDL_MOUSEMOTION){
                                static int i;
                                for(i=0;i<6;i++){
                            		if(punto_en_rect(evento->motion.x,evento->motion.y,&rectConfiguracion[i][MENU_BOTON_CAMBIAR])){
                                        if(botones_cambiar[i]!=BOTON_PRESIONADO)
                                				botones_cambiar[i]=BOTON_RESALTADO;
                                    }
                                    else{
                                        botones_cambiar[i]=BOTON_NORMAL;
                                    }
                                }
    
                        }
    
                    	else if(evento->type == SDL_MOUSEBUTTONDOWN&&evento->button.button==SDL_BUTTON_LEFT) {
                                static int i;
                                for(i=0;i<6;i++)
                            		if(punto_en_rect(evento->motion.x,evento->motion.y,&rectConfiguracion[i][MENU_BOTON_CAMBIAR]))
                                				botones_cambiar[i]=BOTON_PRESIONADO;
    
    
    
    
    
                    	}
                    	else if(evento->type == SDL_MOUSEBUTTONUP&&evento->button.button==SDL_BUTTON_LEFT) {
                                for(int i=0;i<6;i++)
                            		if((botones_cambiar[i]==BOTON_PRESIONADO)&&punto_en_rect(evento->motion.x,evento->motion.y,&rectConfiguracion[i][MENU_BOTON_CAMBIAR])){
                                                    id_espera_tecla=(TeclaPlayer)i;
                                                    botones_cambiar[i]=BOTON_NORMAL;
                                                    game->play(SFX_TONO_ACUATICO);
                                     }
                        }
                        for(int i=0;i<_PLAYERS;i++)
                            botonPlayer[i]->procesarEvento(evento);
                        botonGuardar.procesarEvento(evento);
                    break;
            case VENTANA_CREDITOS://configuracion
                if(evento->type==SDL_KEYDOWN){
                     setDesvanecimiento(-1,VENTANA_1);
                }
                break;
    
    	        }
        }
}


void Menu::draw(SDL_Surface * screen){
    
    if(!animacion){
        SDL_Rect rect={0,0,0,0};
    
        
        switch(ventana){
            case VENTANA_1:case VENTANA_2:
                SDL_BlitSurface(game->getImagen(IMG_FONDO_MENU),NULL, screen,&rect); //Dibujamos el fondo
                for(int i=0;i<5;i++)
                    if(!(ventana==VENTANA_2 && i==4))
                        imprimir_palabra(screen,(i==selected)?game->getImagen(IMG_FUENTE_8):game->getImagen(IMG_FUENTE_7),
                                        rectsImpresion[ventana][i].x,
                                        rectsImpresion[ventana][i].y,
                                        texto[ventana][i],STR_MAX_ESTENDIDA);
                    
                break;
            case VENTANA_3://multijugador
                dibujar_objeto(game->getImagen((CodeImagen)mapa.getIdFondo()),0,0,screen);
                dibujar_objeto(game->getImagen(IMG_TABLERO),0,mapa.getYPanel(),screen);//imprimimos la barra mensage
                dibujar_objeto(game->getImagen(IMG_CUADRO_PEQUENIO),177,7+mapa.getYPanel(),screen);//imprimimos la barra mensage
                dibujar_objeto(game->getImagen(IMG_CUADRO_PEQUENIO),280,7+mapa.getYPanel(),screen);//imprimimos la barra mensage
                dibujar_objeto(game->getImagen(IMG_TXT_PLAYERS_EN_BATALLA),15,24+mapa.getYPanel(),screen);//imprimimos la barra mensage
                dibujar_objeto(game->getImagen(IMG_TXT_TIEMPO_POR_RONDA),140,24+mapa.getYPanel(),screen);//imprimimos la barra mensage
                dibujar_objeto(game->getImagen(IMG_TXT_VICTORIAS),261,24+mapa.getYPanel(),screen);//imprimimos la barra mensage
                
                static char tmp[50];
                
                sprintf(tmp,"%d",minutosEscogidos);
                imprimir_palabra(screen,game->getImagen(IMG_FUENTE_6),178,8+mapa.getYPanel(),tmp,STR_MAX_ESTENDIDA);
                sprintf(tmp,"%d",victoriasEscogidas);
                imprimir_palabra(screen,game->getImagen(IMG_FUENTE_6),280,8+mapa.getYPanel(),tmp,STR_MAX_ESTENDIDA);
    
                btnSubirTiempo->draw(screen);
                btnSubirVictorias->draw(screen);
                mapa.draw(screen);//imprimimos el nivel
                
                sprites->draw(screen);
                for(int i=0;i<_PLAYERS;i++){
                    if(!player_batalla[i]){
                	   imprimir_desde_grilla(game->getImagen((CodeImagen)(IMG_PLAYER_1 + i)), 6,screen, animaPlayer[i]->getX(),animaPlayer[i]->getY(),1, 12,true);
                       sprintf(tmp,"%d",i+1);
                       imprimir_palabra(screen,game->getImagen(IMG_FUENTE_6),animaPlayer[i]->getX()-9+41,animaPlayer[i]->getY()+19,tmp,STR_MAX_ESTENDIDA);
                    }else{
                       imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN),i*2,screen,i*16+20,mapa.getYPanel()+2,1,10,0);
                    }
                }
                btnCambiarMapa->draw(screen);
                btnJugar->draw(screen);
                
                break;
            case VENTANA_4://configuracion
                SDL_BlitSurface(game->getImagen(IMG_FONDO_MENU),NULL, screen,&rect); //Dibujamos el fondo
    
                static char nombre_tecla[20];
                for(int i=0;i<6;i++){
                    //imprimimos el boton
                    imprimir_desde_grilla(game->getImagen(IMG_BOTON_CAMBIAR),(id_espera_tecla==i)?3:botones_cambiar[i],screen,rectConfiguracion[i][MENU_BOTON_CAMBIAR].x,rectConfiguracion[i][MENU_BOTON_CAMBIAR].y,4,1,0);
    
                    //imprimimos la caja de mensaje
                    imprimir_desde_grilla(game->getImagen(IMG_GUI_INPUT_TEXT),id_espera_tecla==i,screen,rectConfiguracion[i][MENU_CUADRO_MOSTRAR].x,rectConfiguracion[i][MENU_CUADRO_MOSTRAR].y,2,1,0);
    
    
                    if(control_edit.isBotonJoystick((TeclaPlayer)i))
                        sprintf(nombre_tecla,"joy %d",control_edit.getKey((TeclaPlayer)i) + 1);
                    else if(control_edit.isDireccionJoystick(i))
                        sprintf(nombre_tecla,"joy %s",SDL_GetKeyName(control_edit.getKey((TeclaPlayer)i)));
                    else if(!strcmp(SDL_GetKeyName(control_edit.getKey((TeclaPlayer)i)),"unknown key")) 
                        strcpy(nombre_tecla,"unknown"); 
                    else
                        strcpy(nombre_tecla,SDL_GetKeyName(control_edit.getKey((TeclaPlayer)i)));
    
                    //imprimimos la tecla
                    imprimir_palabra(screen,game->getImagen(IMG_FUENTE_6),rectConfiguracion[i][MENU_CUADRO_MOSTRAR].x,rectConfiguracion[i][MENU_CUADRO_MOSTRAR].y,nombre_tecla,STR_MAX_ESTENDIDA);
    
                    //imprimimos el texto
                    imprimir_desde_grilla(game->getImagen((CodeImagen)(IMG_TXT_ARRIBA + i)),id_espera_tecla==i,screen,rectConfiguracion[i][MENU_TEXTO_MOSTRAR].x,rectConfiguracion[i][MENU_TEXTO_MOSTRAR].y,2,1,0);
                }
                for(int i=0;i<_PLAYERS;i++)
                     botonPlayer[i]->draw(screen);
                     
    
    
                botonGuardar.draw(screen);
                imprimir_desde_grilla(game->getImagen(IMG_CARAS_BOMBERMAN),player_configurando_teclas*2,screen,rect_destino_cara.x,rect_destino_cara.y,1,10,0);
    
    
                break;
            case VENTANA_CREDITOS://configuracion
                dibujar_objeto(game->getImagen(IMG_FONDO_CREDITOS),0,0,screen);
                break;
            }
        }else{
            if(nAnimacion==1)
                dibujar_objeto(fondoVentanaAnterior,0,0,screen);
            else if(nAnimacion==2)
                dibujar_objeto(fondoVentanaSiguiente,0,0,screen);
            dibujar_objeto(fondoNegro,0,0,screen);
      }

}



Menu::~Menu(){
    #ifdef DEBUG
    cout << "Destructor de Menu:"<<this<<endl;
    #endif
    for(int i=0;i<_PLAYERS;i++){
        delete botonPlayer[i];
        delete animaPlayer[i];
        delete animaPresiona[i];
        delete animaActivado[i];
    }
    SDL_FreeSurface(previewTerreno);
    delete dataNivel;
    delete sprites;
    delete btnSubirTiempo;
    delete btnSubirVictorias;
    delete btnCambiarMapa;
    delete btnJugar;
    SDL_FreeSurface(fondoVentanaAnterior);
    SDL_FreeSurface(fondoVentanaSiguiente);
    SDL_FreeSurface(fondoNegro);
//    delete fuente7; 
}


// PLAYER 

#include "player.hpp"


//#define DEBUG

Player::Player(Juego * juego,IdPlayer id,int x,int y,int vidasIni,int numBombasIni,int alcanceBombasIni){
    #ifdef DEBUG
    cout << "Constructor de Player:"<<this<<endl;
    #endif

    this->juego=juego;

    this->id=id;
    
    rect.w=W_COLISION;
    rect.h=H_COLISION;

    mantieneStartPresionado=false;
    self_kill=false;
	estaProtegido=false;

    idUltimaBomba=-1;

    //variables para reiniciarlo
    this->alcanBombIni=alcanceBombasIni;
    this->numBombasIni=numBombasIni;

    xIni=x;
    yIni=y;

    this->vidas=vidasIni;


	estado = ABAJO;//estado del personaje que pasara a ser el anterior(solo para que mire para abajo)
    reiniciar();
//	cambiarEstado(PARADO);
    cargarTeclas();
//	move(x,y);

}

void Player::reiniciar(){
    enPantalla=true;
    muerto=false;
    entroPuerta=false;
    puedeAtravesarBloquesBlandos=false;
    alcanBomb=alcanBombIni;
    numBombas=numBombasIni;
    velocidad=1;
    puedeAtravesarBombas=false;
    puedeGolpearBombas=false;
    estaEnfermo=false;
    corazones=0;
    puntaje=0;
    cambiarEstado(PARADO);
    estaProtegido=false;
    move(xIni,yIni);
//    juego->resetEjes();
//    setProteccion(10);

}

void Player::disable(){
	enPantalla=false;
	juego->erase(PLAYER,getId());
}

void Player::cargarTeclas(){
    char tmp_ruta[40];
    sprintf(tmp_ruta,"data/configuracion/teclado_%d.dat",id+1);

    if(!control.cargar(tmp_ruta,false)){//si no se puede cargar de un archivo
        //se asignan teclas por default
        control.setDefaultKeys((IdPlayer)id);
    }
}



void Player::updateRectColision(){
    //actualiza el cuadro que representa al personaje en la colision
    rect.x=x+X_COLISION;
    rect.y=y+Y_COLISION;

}


void Player::update(Uint8 * teclas){
    
	avanzarAnimacion ();//avanzamos la animacion
    updateRectColision();
	switch (estado){
		case PARADO:
		    parado (teclas);
			break;

		case IZQUIERDA:
			izquierda(teclas);
			break;

		case DERECHA:
			derecha(teclas);
			break;

		case ARRIBA:
			arriba(teclas);
			break;
		case ABAJO:
			abajo(teclas);
			break;
		case MURIENDO:break;
	}

    if(estado!=MURIENDO){
        if(!estaProtegido){//si no esta protegido
            static int id_explo;
            if(juego->colision(GLOBO,rect)!=-1){
                if(!corazones)
                    cambiarEstado(MURIENDO);
                else{
                    setProteccion(10);
                    corazones--;
                }
            }else if((id_explo=juego->colision(EXPLOSION,rect))!=-1){
                if(!corazones){
                    cambiarEstado(MURIENDO);
                    if(juego->getTipoJuego()==TIPO_BATALLA){
                        /*if(juego->explosiones[id_explo-1]->lanzador!=id)
                            juego->matadas[personajes->juego->objetos->explosiones[id_explo-1]->lanzador]++;
                        else
                            juego->matadas[personajes->juego->objetos->explosiones[id_explo-1]->lanzador]--;
                        juego->kills[id]++;*/
                        if(juego->getLanzador(EXPLOSION,id_explo)!=this->id);
    //                        juego->matadas[personajes->juego->objetos->explosiones[id_explo-1]->lanzador]++;
                        else;
    //                        juego->matadas[personajes->juego->objetos->explosiones[id_explo-1]->lanzador]--;
    //                    juego->kills[id]++;
                        
                    }//fin "si tipo de juego es BATALLA"
                }else{
                    setProteccion(10);
                    corazones--;
                }
            }//fin "si el id de la explosion no es -1"
        }else{//si esta protegido
            if(juego->getTick()-tiempoInicioProteccion>=duracionProteccion){
                estaProtegido=false;
            }
        }
    }//fin "si no esta muriendo"

    /*SI COLISIONA CON ALGÚN ITEM*/
      int id_item;//almacena la respuesta de: ¿con que item esta colisionando?

      id_item=juego->colision(ITEM,rect);
      if(id_item!=-1){//¿colisiono con uno?
            int tipo_item=juego->getTipoItem(id_item);
            if(tipo_item==Item::ITEM_PUERTA){
                if(juego->getActivos(GLOBO)==0)
                    entroPuerta=true;
            }else{
                setPuntaje(getPuntaje()+20);
                activarPoderItem(tipo_item);
                juego->killSprite(ITEM,id_item);
                juego->play(SFX_COGER_ITEM);
            }
       }

    /*SI EL PLAYER ESTA MUERTO*/
    if(muerto){//si el player esta muerto
        if(--vidas>=0){//si sigue con vida
            reiniciar();
            setProteccion(5);
            juego->play(SFX_PIERDE_VIDA);
        }
        else
            disable();
    }


    if (entroPuerta&&juego->getTipoJuego()==TIPO_NORMAL){//si entro en la puerta de fin de nivel
            /*entro_puerta=false;
            proteccion=true;
            cambiarEstado(PARADO);
            int bonus=time(0)-juego->getSegundosInicioNivel();
            setPuntaje(getPuntaje()+bonus);
            if(juego->isActivo(PLAYER,PLAYER_2)){
                juego->setPuntaje(PLAYER_2,juego->getPuntaje(PLAYER_2) + bonus);
                juego->cambiarEstadoPlayer(PLAYER_2,PARADO);
            }
            juego->setNivelPlay(juego->getNivelActual() + 1,false);*/
            juego->aumentarNivel();
            
    }
}

void Player::activarPoderItem(int tipo){
//    cout <<"activando poder item:"<<tipo<<endl;
    switch(tipo){
        case Item::ITEM_ALCANCE:
            if(alcanBomb < MAX_ALCANCE_EXPLOSION)
                alcanBomb++;
            break;
        case Item::ITEM_VIDA:
            vidas++;
            break;
        case Item::ITEM_BOMBA:
            if(numBombas < MAX_BOMBAS)
                numBombas++;
            break;
        case Item::ITEM_ATRAVIESA_PAREDES:
            puedeAtravesarBloquesBlandos=true;
            break;
        case Item::ITEM_ALEATORIO:
            activarPoderItem(juego->getTipoNuevoItem(false));
            break;
        case Item::ITEM_BOMBA_MAX:
            numBombas=MAX_BOMBAS;
            break;
        case Item::ITEM_ALCANCE_MAX:
            alcanBomb=MAX_ALCANCE_EXPLOSION;
            break;
        case Item::ITEM_PROTECCION:
            setProteccion(20);
            break;
        case Item::ITEM_BOLA_ARROZ:
            setPuntaje(getPuntaje()+100);
            break;
        case Item::ITEM_PASTEL:
            setPuntaje(getPuntaje()+120);
            break;
        case Item::ITEM_PALETA:
            setPuntaje(getPuntaje()+120);
            break;
        case Item::ITEM_BARQUILLA:
            setPuntaje(getPuntaje()+50);
            break;
        case Item::ITEM_MANZANA:
            setPuntaje(getPuntaje()+250);
            break;
        case Item::ITEM_PATINETA:
            velocidad=2;
            break;
        case Item::ITEM_CORAZON:
            corazones++;
            break;
        case Item::ITEM_ATRAVIESA_BOMBAS:
            puedeAtravesarBombas=true;
            break;
        case Item::ITEM_PATEA_BOMBA:
            puedePatearBombas=true;
            break;
        default:
            break;
        }
}

void Player::draw(SDL_Surface * screen){
    if(estado!=MURIENDO)
    	imprimir_desde_grilla (juego->getImagen((CodeImagen)(IMG_PLAYER_1 + id)), cuadro,screen, x,y,1, 12,estaProtegido);
    else
    	imprimir_desde_grilla(juego->getImagen((CodeImagen)(IMG_PLAYER_1_MURIENDO + id)), cuadro,screen,x,y,1, 4,0);
//    if(estaProtegido)dibujar_objeto(juego->getImagen(IMG_FONDO_BLANCO),x,y,screen);
	/*DIBUJA EL CUADRO QUE REPRESENTA LA COLISION DEL PERSONAJE*/
#ifdef DEBUG
            updateRectColision();
            SDL_FillRect(screen,&rect,SDL_MapRGB (screen->format, 0, 0, 255));
#endif
}


void Player::ponerBomba(Uint8 * teclas){
    TipoSprite conjun_coli[]={BOMBA,GLOBO,NIVEL,ITEM};
    
    SDL_Rect rect_bomb={0,0,16,16};
    updateRectColision();
	if(!mantieneStartPresionado&&\
        isPressed(TECLA_ACCION,teclas)&&\
        (juego->getActivosId(BOMBA,(IdPlayer)id) < numBombas)&&\
        (juego->colision(conjun_coli,4,rect)==-1)){

                /*anyadimos la bomba inocentemente*/
               int id_bomba_colocada=juego->addSprite(BOMBA,(x+7-juego->getEjeXVisual())/16*16+juego->getEjeXVisual(),(y+11-juego->getEjeYVisual())/16*16+juego->getEjeYVisual(),(int)id);
               /*si se logro anyadir*/
    	       if(id_bomba_colocada!=-1){
                    /*si la bomba que colocamos colisiona con un personaje la quitamos porque el PLAYER CON EL QUE COLISIONA
                    NO SE MOVERA NUNCA*/
                    int x,y;
                   juego->getPosicion(BOMBA,id_bomba_colocada,x,y);
                   rect_bomb.x=x;
                   rect_bomb.y=y;
    	           if(juego->colision(PLAYER,rect_bomb,id)!=-1||juego->colision(GLOBO,rect_bomb)!=-1){
                            juego->soloKill(BOMBA,id_bomba_colocada);
                            return;
                    }
                    /*sino ocurre lo de arriba continuamos  */
                   idUltimaBomba=id_bomba_colocada;
                }
        }

	mantieneStartPresionado=isPressed(TECLA_ACCION,teclas);
}

bool Player::colision(SDL_Rect & rect){

    updateRectColision();
    return rects_colisionan(this->rect,rect);
}

void Player::parado(Uint8 * teclas)
{
//     if(!es_bot){
        if(isPressed(TECLA_ARRIBA,teclas))
            cambiarEstado(ARRIBA);
    
        if(isPressed(TECLA_ABAJO,teclas))
            cambiarEstado (ABAJO);
    
        if(isPressed(TECLA_IZQUIERDA,teclas))
            cambiarEstado (IZQUIERDA);
    
        if(isPressed(TECLA_DERECHA,teclas))
            cambiarEstado(DERECHA);

        ponerBomba(teclas);
    /*}else{
       static int delay;
       static int estados
          if(++delay>=100){
               delay=0;
               
          }
    }   */


}

void Player::izquierda(Uint8 * teclas)
{
	mover_ip(-velocidad,0);
	if(!isPressed(TECLA_IZQUIERDA,teclas))
    	cambiarEstado(PARADO);

	ponerBomba(teclas);


}

void Player::derecha (Uint8 * teclas)
{
	mover_ip(velocidad,0);
	if(!isPressed(TECLA_DERECHA,teclas))
    	cambiarEstado ( PARADO);

	ponerBomba(teclas);
}


bool Player::isPressed(TeclaPlayer tecla, Uint8 * _teclas){
    if(!control.isBotonJoystick(tecla) && !control.isDireccionJoystick(tecla)){
        return _teclas [control.getKey(tecla)];

    }else{
        for(int i=0;i<juego->getJoysActivos();i++){
            if(!strcmp(SDL_JoystickName(i),control.getName(tecla))){//si coincide con el joistick con el que se configuro
				return estado_tecla_joy(control.getKey(tecla),juego->getJoy(i));
             }
         }
         return false;
     }
}
void Player::arriba (Uint8 * teclas)
{
    mover_ip (0,-velocidad);
	if(!isPressed(TECLA_ARRIBA,teclas))
    	cambiarEstado(PARADO);

	ponerBomba(teclas);


}

void Player::abajo(Uint8 * teclas)
{
	mover_ip(0,velocidad);
	if(!isPressed(TECLA_ABAJO,teclas))
    	cambiarEstado (PARADO);

	ponerBomba(teclas);


}

/*
 * modifica el cuadro de la animación que se debe mostrar en pantalla
 */
void Player::avanzarAnimacion ()
{
	static int animaciones [_ESTADOS][17] = {\
		{3,3, 4,4, 5,5,-1},\
		{9,9, 10,10, 11,11, -1},\
		{6,6,  7,7, 8,8,-1},\
		{0,0, 1,1, 2,2,-1},\
        {0,0,0,0,1,1,1,1,2,2,2,2,3,3,3,3,-1}};
	if (--delay < 1)
	{
		delay = 3;

		if (animaciones [estado] [paso + 1] == -1){
		    if(estado!=MURIENDO)
    			paso = 0;
    		else
    		   muerto=1;
		}else
			paso ++;
	}


	if(estado!=PARADO)
	   cuadro = animaciones [estado][paso];
	else
	   cuadro = animaciones [estado_anterior][0];

}

void Player::cambiarEstado(EstadoSprite nuevo)
{
    estado_anterior=estado;
	estado = nuevo;
	paso = 0;
	delay = 3;
}

void Player::setProteccion(int segundos){
    duracionProteccion=segundos;
    tiempoInicioProteccion=juego->getTick();
    estaProtegido=true;
}
void Player::mover_ip(int incremento_x, int incremento_y)
{//mueve al personaje detectando alguna colision
    int temp,num_colision=0;

    rect.x+=incremento_x;
    rect.y+=incremento_y;
    temp=juego->colision(BOMBA,rect);

    if(temp!=-1&&temp!=idUltimaBomba&&!puedeAtravesarBombas){ /*si esta sobre una bomba que no es la que el puso*/
        return;
     }else if(temp==-1&&idUltimaBomba!=-1){
        idUltimaBomba=-1;
    }    

    
    if(rect.x<juego->getEjeXVisual()||\
       rect.x+rect.w>juego->getEjeXVisual()+juego->getAnchoMapa()||\
       rect.y<juego->getEjeYVisual()||\
       rect.y+rect.h>juego->getEjeYVisual()+juego->getAltoMapa())return;

    if(!puedeAtravesarBloquesBlandos)
        temp=juego->colision(rect,&num_colision,false);
    else
        temp=juego->colision(rect,&num_colision,true);
    
    if(temp){
        if(num_colision==1){//ESTO ES PARA DESPLAZAR EL PERSONAJE UN POCO
            if(estado==IZQUIERDA||estado==DERECHA){
                if(temp==1||temp==2)
                    y-=1;
                else
                    y+=1;
            }else{
                if(temp==1||temp==4)
                    x-=1;
                else
                    x+=1;
            }
        }
    }else{
        move(x+incremento_x,y+incremento_y);
    }

}
void Player::move(int x,int y){
    //establece al jugador en la posicion indicada

    this->x=x;
    this->y=y;
    if(x>W_SCREEN/3*2&&estado==DERECHA){
        if(juego->moveLeftEjeXVisual()){
            this->x=W_SCREEN/3*2;
        }
        
    }
    if(x<W_SCREEN/3&&estado==IZQUIERDA){
        if(juego->moveRightEjeXVisual()){
            this->x=W_SCREEN/3;
        }
    }
}

Player::~Player(){
    #ifdef DEBUG
    cout << "Destructor de Player:"<<this<<endl;
    #endif
}


// UTIL 

#include "util.hpp"

template <typename T> bool dato_en_array(T dato,T array[],int N){
    int i;
    
     for(i=0;i<N;i++)
        if(array[i]==dato)
           return true;
           
     return false;
     
}

void sort_array(int array_sort[5],int destino_sort[5]){
     int i,index,mayores[5];

     for(i=0;i<5;i++){
        destino_sort[i]=-1000;
        mayores[i]=-1000;
     }
        
     
     for(index=0;index<5;index++){
       for(i=0;i<5;i++){
             if(array_sort[i]>mayores[index]&&!dato_en_array(i,destino_sort,5)){
                destino_sort[index]=i;
                mayores[index]=array_sort[i];
             }
       }
    }
}

SDL_Surface *cargar_imagen(char ruta[],bool keyclave){
	SDL_Surface * imagen;

	imagen = IMG_Load (ruta);

	if (!imagen){
		cout<< "No se puede cargar:"<< ruta<<endl;
		exit(1);
	}
    
    if(keyclave){	
	   SDL_SetColorKey (imagen, SDL_SRCCOLORKEY, \
				SDL_MapRGB (imagen->format, 0, 255, 0));
	}
	
    cout<< "+ cargando:"<< ruta<<endl;
    return imagen;
}

void mostrar_error(char msg[]){
    cerr << "Error:"<<msg<<"Error SDL:"<<SDL_GetError()<<endl;
    exit(1);
}

Mix_Chunk * cargar_sonido(char ruta[]){
    Mix_Chunk * cargado;
    cargado=Mix_LoadWAV(ruta);
    if(cargado==NULL){
        cerr<<"Error cargando sonido:"<<ruta<<Mix_GetError()<<endl;
        exit(1);
    }
	cout << "+ cargando:"<<ruta<<endl;
	return cargado;
}
Mix_Music * cargar_musica(const char ruta[]){
    Mix_Music * cargado;
    cargado=Mix_LoadMUS(ruta);
    if(cargado==NULL){
        cerr<<"Error cargando musica:"<<ruta<<Mix_GetError()<<endl;
        exit(1);
    }
	cout << "+ cargando:"<<ruta<<endl;
	return cargado;
}
bool rects_colisionan(SDL_Rect & rect_1,SDL_Rect & rect_2)
{
    
    return (((rect_1.x+rect_1.w)>rect_2.x) && \
    ((rect_1.y+rect_1.h)>rect_2.y) &&\
    ((rect_2.x+rect_2.w)>rect_1.x) && \
    ((rect_2.y+rect_2.h)>rect_1.y)) ;
}


void imprimir_desde_grilla(SDL_Surface * src, int cuadro, SDL_Surface *dst,int x_dest,int y_dest, int fil, int col,int alpha)
{
	SDL_Rect srcrect,dest_rect={x_dest,y_dest,0,0};

	srcrect.w = src->w / col;
	srcrect.h = src->h / fil;
	srcrect.x = (cuadro % col) * srcrect.w;
	srcrect.y = (cuadro / col) * srcrect.h;

	if(alpha)
    	SDL_SetAlpha(src, SDL_SRCALPHA|SDL_RLEACCEL,150);
    else
    	SDL_SetAlpha(src, SDL_SRCALPHA|SDL_RLEACCEL, 255);

	SDL_BlitSurface(src, &srcrect, dst, &dest_rect);
}


int fps_sincronizar (void)
{
	static int t;
	static int tl = 0;
	static int frecuencia = 1000 / 70;
	static int tmp;

	t = SDL_GetTicks ();

	if (t - tl >= frecuencia)
	{
		tmp = (t - tl) / frecuencia;
		tl += tmp * frecuencia;
		return tmp;
	}
	else
	{
		SDL_Delay (frecuencia - (t - tl));
		tl += frecuencia;
		return 1;
	}

}


/*
 * Relaciona un caracter con un número entero
 */
int obtener_indice (char caracter,char * orden_letras)
{
	int i;
				
	for (i = 0; orden_letras [i]; i ++)
	{
		if (caracter == orden_letras [i])
			return i;
	}
	
	return -1;
}


/*
 * imprime un caracter sobre la superficie dst (generalmente screen)
 */
int imprimir_letra (SDL_Surface * dst, SDL_Surface * ima,int x, int y, char letra,char * orden_letras)
{
	SDL_Rect srcrect;
	SDL_Rect dstrect = {x, y, 0, 0};
	
	int cantidad_de_letras=strlen(orden_letras);
	
	srcrect.w=ima->w/cantidad_de_letras;
	srcrect.x=srcrect.w*obtener_indice(letra,orden_letras);
	srcrect.y=0;
	srcrect.h=ima->h;
	if(srcrect.x>=0)
    	SDL_BlitSurface(ima, &srcrect, dst, &dstrect);
	
	return ima->w/cantidad_de_letras;
}


/*
 * imprime una cadena de textos completa sobre la superficie referenciada
 * por el primer parámetro
 */
void imprimir_palabra (SDL_Surface * screen, SDL_Surface * ima, int x, int y,const char * cadena,char * orden_letras)
{
	int i;
	int dx = x;

	for (i = 0; cadena [i]; i ++)
		dx += imprimir_letra (screen, ima, dx, y, cadena [i],orden_letras);
}
void mostrar_msg (SDL_Surface * screen, SDL_Surface * ima, int x,int y,const char * orden_letras, char * formato, ...)
{
    va_list lista;
    char buffer [1024];
    va_start (lista, formato);
        vsprintf (buffer, formato, lista);
        imprimir_palabra (screen, ima, x, y,orden_letras, buffer);
    va_end (lista);
}

Uint32 get_pixel (SDL_Surface * ima, int x, int y)
{
	int bpp = ima->format->BytesPerPixel;
	Uint8 *p = (Uint8 *) ima->pixels + y * ima->pitch + x * bpp;

	switch (bpp)
	{
		case 1:
			return *p;
		
		case 2:
			return *(Uint16 *)p;

		case 3:
			if (SDL_BYTEORDER == SDL_BIG_ENDIAN)
				return p[0] << 16 | p[1] << 8 | p[2];
			else
				return p[0] | p[1] << 8 | p[2] << 16;

		case 4:
			return *(Uint32 *)p;

		default:
			return 0;
	}
}


int buscar_dato(char * ruta,char * nombre_dato){
    static FILE *fscript;
    
    int valor;
    char linea[100],*identificador;
    
    if(!(fscript=fopen(ruta,"r"))){
      sprintf(linea,"Error leyendo archivo(Buscar Dato):%s\n",ruta);
      mostrar_error(linea);
    }

    while(!feof(fscript)){    
        fgets(linea,100,fscript);
        
        identificador = strtok(linea , ":") ;
        if(!strcmp(identificador,nombre_dato)){
            sscanf(strtok( NULL , " "),"%d",&valor);
            fclose(fscript);
            return valor;
        }
    }
    fclose(fscript);
    return -1;
}

bool estado_tecla_joy(SDLKey tecla,SDL_Joystick * joy){
	switch(tecla){
		case SDLK_LEFT:
				return SDL_JoystickGetAxis(joy, 0) < -10;
			break;
		case SDLK_RIGHT:
				return SDL_JoystickGetAxis(joy, 0) > 10;
			break;
		case SDLK_UP:
				return SDL_JoystickGetAxis(joy, 1) <-10;
			break;
		case SDLK_DOWN:
				return SDL_JoystickGetAxis(joy, 1) > 10;
			break;
		default:
			return SDL_JoystickGetButton(joy, tecla);
	}
}
void dibujar_objeto(SDL_Surface *src,Sint16 x,Sint16 y,SDL_Surface *dst){

    if(src==NULL||dst==NULL){
       cerr<<"WARNING:intento de dibujado en funcion:dibujar_objeto sobre o con valor NULL"<<endl;

       return;
    }

    SDL_Rect dest={x,y,src->w,src->h};
    SDL_BlitSurface(src,NULL,dst,&dest);

}



EstadoSprite invertir_estado(EstadoSprite estado){
    switch(estado){
        case DERECHA:
            return IZQUIERDA;
        case IZQUIERDA:
            return DERECHA;
        case ARRIBA:
            return ABAJO;
        case ABAJO:
            return ARRIBA;
        default:
            printf("no implementado para ese estado:%d\n",estado);
            return ABAJO;
        }
}

void sdl_videoinfo(void)
{

    const SDL_VideoInfo *propiedades;
    SDL_Surface *pantalla;
    SDL_Rect **modos;

    //Variables auxiliares
    char driver[20];
    int maxlen = 20;
    int i = 0;

    // Obtenemos la informaciÃ³n del sistema de video
    propiedades = SDL_GetVideoInfo();
    if(propiedades == NULL) {
	 fprintf(stderr, "No se pudo obtener la informaciÃ³n %s\n",
		  SDL_GetError());
	 exit(1);
    }

    // Obtenemos los modos de video disponibles
    modos = SDL_ListModes(NULL, SDL_HWSURFACE);

    printf("\n\n == MODOS DE VIDEO DISPONIBLES == \n");

    // Comprobamos que mÃ©todos estÃ¡n disponibles
    if(modos == (SDL_Rect **)0)
	 printf("No existen modos disponibles \n");
    else if(modos == (SDL_Rect **)-1)
	 printf("Todos los modos disponibles \n");
    else {
	 printf("Lista de modos disponibles\n");
	 for(i = 0; modos[i]; i++)
	     printf("%d x %d\n", modos[i]->w, modos[i]->h);
    }

    // Comprobamos que el modo a seleccionar sea compatible
    if(SDL_VideoModeOK(640, 480, 24, SDL_SWSURFACE) == 0) {
	 fprintf(stderr, "Modo no soportado: %s\n", SDL_GetError());
	 exit(1);
    }

 
    // Una vez comprobado establecemos el modo de video
    pantalla = SDL_SetVideoMode(640, 480, 24, SDL_SWSURFACE);
    if(pantalla == NULL)
	 printf("SDL_SWSURFACE 640x480x24 no compatible. Error: %s\n",
		 SDL_GetError());


    // Obtenemos informaciÃ³n del driver de video
    printf("\n\n == INFORMACIÃ“N DRIVER VIDEO == \n");
    SDL_VideoDriverName(driver, maxlen);

    if(driver == NULL) {
	 fprintf(stderr, "No se puede obtener nombre driver video\n");
	 exit(1);
    }

    printf("Driver: %s\n", driver);

    
    // Obtenemos informaciÃ³n sobre las capacidades de nuestro
    // sistema respecto a SDL
    printf("\n == INFORMACION SDL_INFO == \n\n");
    if(propiedades->hw_available == 1)
	 printf("HW Compatible\n");
    else
	 printf("HW no compatible\n");

    if(propiedades->wm_available == 1)
	 printf("Hay un manejador de ventanas disponible\n");
    else
	 printf("No hay un manejador de ventanas disponible\n");

    if(propiedades->blit_hw == 1)
	 printf("El blitting hardware - hardware estÃ¡ acelerado\n");
    else
	 printf("El blitting hardware - hardware NO estÃ¡ acelerado\n");

    if(propiedades->blit_hw_CC == 1) {
	 printf("El blitting con transparencias hardware - hardware ");
	 printf("estÃ¡ acelerado\n");
    }
    else {
	 printf("El blitting con transparencias hardware - hardware ");
	 printf("NO estÃ¡ acelerado\n");
    }

    if(propiedades->blit_sw == 1)
	 printf("El blitting software - hardware estÃ¡ acelerado.\n");
    else
	 printf("El blitting software - hardware NO estÃ¡ acelerado. \n");
    
    if(propiedades->blit_sw_CC == 1) {
	 printf("El blitting software - hardware con transparencias");
	 printf(" estÃ¡ acelerado\n");
    }
    else {
	 printf("El blitting software - hardware con transparencias");
	 printf(" NO estÃ¡ acelerado\n");
    }

    if(propiedades->blit_sw_A == 1)
	 printf("El blitting software - hardware con alpha estÃ¡ acelerado\n");
    else
	 printf("El blitting software - hardware con alpha NO estÃ¡ acelerado\n");

    if(propiedades->blit_fill == 1)
	 printf("El rellenado de color estÃ¡ acelerado\n");
    else
	 printf("El rellenado de color NO estÃ¡ acelerado\n");

    printf("La memoria de video tiene %f MB\n", (float) propiedades->video_mem);
				
}


