#include <raylib.h>
#include <stdlib.h>
#include <stdio.h>
#include <time.h>
#include <math.h>
#include <unistd.h>
#include <stdbool.h>
#include <string.h>

//----------------------------------------------------------------------------------
// Some Defines
//----------------------------------------------------------------------------------
#define STD_SIZE_X 25
#define STD_SIZE_Y 25
#define STD_SIZE_BOMB_X 25
#define STD_SIZE_BOMB_Y 25
#define SCREEN_BORDER 40

//----------------------------------------------------------------------------------
// Texturas e Sons
//----------------------------------------------------------------------------------

Texture2D mario_costas;
Texture2D mario_esquerda;
Texture2D mario_direita;
Texture2D mario_frente;
Texture2D luigi_costa;
Texture2D luigi_esquerda;
Texture2D luigi_direita;
Texture2D luigi_frente;
Texture2D cenario1;
Texture2D cenario2;
Texture2D barreiras;
Texture2D barreiras_1;
Texture2D barreiraquebravel;
Texture2D barreiralateral;
Texture2D tela_inicial;
Texture2D escolhemapa;
Texture2D tela_revanche;
Texture2D game_over;
Texture2D inimigo_1;
Texture2D inimigo_2;
Texture2D inimigo_3;
Texture2D inimigo_4;
Texture2D bomba;
Texture2D velocidade;
Texture2D raio;
Texture2D barreiracanto;
Texture2D tela_comeco;

Music musicaFim;
Music musicaInicio;
Music musicaGame;
Music sound_bomba;


typedef struct Bomb
{
    Rectangle pos;
    Rectangle explosion_right;
    Rectangle explosion_left;
    Rectangle explosion_up;
    Rectangle explosion_down;
    int isActive;
    int distance;
    double time;
    int local;
} Bomb;

typedef struct Hero
{
    char name[100];
    Rectangle pos;
    Color color;
    int speed;
    int special;
    int draw_bomb;
    int put_bomb;
    int num_bombs;
    Bomb bombs[10];
    int dead;
    int vitorias;
    int derrotas;
} Hero;

typedef struct Enemy
{
    Rectangle pos;
    Color color;
    int speed;
    int direction;
    int draw_enemy;
} Enemy;

typedef struct {
    Rectangle rect;
    Color color;
    int isBroken;
    int x, y;
} BreakableBlock;

typedef struct especial{
    int foiUsado;
    Rectangle spc;
    Color color;
}especial;

typedef struct Map
{
    especial esp1[4];
    especial esp2[4];
    especial esp3[4];
    Rectangle barriers[40];
    BreakableBlock blocks[17];
    int num_barriers;
    Color color;
    int next_map;
    int prev_map;
    int screenWidth;
    int screenHeight;
    int pos;
} Map;

typedef struct Game
{
    Map maps[10];
    int num_maps;
    int curr_map;
    Hero hero1;
    Hero hero2;
    int screenWidth;
    int screenHeight;
    int gameover;
    int map;
    Enemy enemies;
    Enemy enemies2;
    int num_enemies;
    especial esp1[4];
    Map m;
    especial esp2[4];
    especial esp3[4];
    Rectangle barriers[40];
    Rectangle borda[4];
    int block;
    int pos;
    int res;
} Game;

//------------------------------------------------------------------------------------
// Global Variables Declaration
//------------------------------------------------------------------------------------
static int framesCounter = 7200;
int nomeEscolhido;
int revancheTop;
//------------------------------------------------------------------------------------
// Module Functions Declaration (local)
//------------------------------------------------------------------------------------
void InitGame(Game *g);        // Initialize game
void UpdateGame(Game *g);      // Update game (one frame)
void DrawGame(Game *g);        // Draw game (one frame)
void UpdateDrawFrame(Game *g); // Update and Draw (one frame)

//------------------------------------------------------------------------------------
// Auxiliar Functions Declaration
//------------------------------------------------------------------------------------
void draw_borders(Game *g);
void DrawBlocks(Game *g);
void DrawSpecial(Game *g);
void draw_map(Game *g);
void draw_bomb(Game *g);
void update_hero1_pos(Game *g);
void update_hero2_pos(Game *g);
void update_enemy_pos(Game *g);
void update_enemy2_pos(Game *g);
void update_bomb(Game *g);
void escolheMap(Game *g);
void nameScreen(Game *g,Hero *h,int n);
int revanche(Game *g);
void placar_final(Game *g);
void TelaInicial(Game *g);

void UpdatePosGame(Game *g);
int barrier2_collision(Map *map, Rectangle target);
int barrier_collision(Map *m, Rectangle t);
int bomb_collision(Hero *h, Rectangle target);
int hero_collision(Rectangle h1, Rectangle h2 );
int CheckCollisionWithBlocks(Map *map, Hero *h);
int bombs_hipercollision(Hero *h,Rectangle *target);
int PegouEspecial1(Hero *h, Game *g);
int PegouEspecial2(Hero *h, Game *g);
int PegouEspecial3(Hero *h, Game *g);
void map0_setup(Game *g);
void map1_setup(Game *g);

//------------------------------------------------------------------------------------
// Program main entry point
//------------------------------------------------------------------------------------
int main(void)
{
    nomeEscolhido = 0;
    revancheTop = 0;
    Game game;
    game.hero1.derrotas=0;
    game.hero2.derrotas=0;
    game.hero1.vitorias=0;
    game.hero2.vitorias=0;
    game.screenWidth = 800;
    game.screenHeight = 600;

    InitWindow(game.screenWidth, game.screenHeight, "Super BombMario World");
    
    mario_costas = LoadTexture("imagens/mario_costas.gif");
    mario_frente = LoadTexture("imagens/mario_frente.gif");
    mario_esquerda = LoadTexture("imagens/mario_esquerda.gif");
    mario_direita = LoadTexture("imagens/mario_direita.gif");
    luigi_costa = LoadTexture("imagens/luigi_costa.gif");
    luigi_frente = LoadTexture("imagens/luigi_frente.gif");
    luigi_esquerda = LoadTexture("imagens/luigi_esquerda.gif");
    luigi_direita = LoadTexture("imagens/luigi_direita.gif");
    cenario1 = LoadTexture("imagens/cenario1.png");
    cenario2 = LoadTexture("imagens/cenario2.png");
    barreiras = LoadTexture("imagens/barreiras.png");
    barreiras_1 = LoadTexture("imagens/barreiras_1.png");
    barreiraquebravel = LoadTexture("imagens/barreiraquebravel.png");
    barreiralateral = LoadTexture("imagens/barreiralateral.png");
    tela_inicial = LoadTexture("imagens/telainicial.png");
    tela_comeco = LoadTexture("imagens/tela_começo.png");
    escolhemapa = LoadTexture("imagens/escolhemapa.png");
    tela_revanche = LoadTexture("imagens/tela_revanche.png");
    inimigo_1 = LoadTexture("imagens/inimigo_1.gif");
    inimigo_2 = LoadTexture("imagens/inimigo_2.gif");
    bomba = LoadTexture("imagens/bomba.png");
    game_over = LoadTexture("imagens/game_over.png");
    velocidade = LoadTexture("imagens/velocidade.png");
    raio = LoadTexture("imagens/raio.png");
    barreiracanto = LoadTexture("imagens/barreiracanto.png");

    SetTargetFPS(120);
    
    SetTargetFPS(120);

    InitAudioDevice();
   
    musicaInicio = LoadMusicStream("sounds/musicaInicio.mp3");
    musicaFim = LoadMusicStream("sounds/musicaFim.mp3");
    musicaGame = LoadMusicStream("sounds/musicaGame.mp3");
    sound_bomba = LoadMusicStream("sounds/sound_bomba.mp3");

    PlayMusicStream(sound_bomba);

    TelaInicial(&game);    

    if(game.res==1){
    InitGame(&game);
    while (!WindowShouldClose()) // Detect window close button or ESC key
    {
        UpdateDrawFrame(&game);
        if (game.gameover){
         int res = revanche(&game);
            if(res==1){
                revancheTop = 1;
                InitGame(&game);
                UpdateDrawFrame(&game);
            }else{
              break;
            }
        }
        }
    }

    while (!IsKeyDown(KEY_ENTER))
    {
        BeginDrawing();
        ClearBackground(RAYWHITE);
        PlayMusicStream(musicaFim);
        UpdateMusicStream(musicaFim);
        DrawTexture(game_over, 0, 0, WHITE);
        if (game.hero1.vitorias > game.hero2.vitorias)
        {
            DrawText(TextFormat("WINNER PLAYER 1: %s!",game.hero1.name), GetScreenWidth() / 2 - MeasureText("WINNER PLAYER 1", 10) / 2, GetScreenHeight() / 2 - 20, 10, BLACK);
        }
        else if (game.hero2.vitorias > game.hero1.vitorias)
        {
            DrawText(TextFormat("WINNER PLAYER 2: %s!",game.hero2.name), GetScreenWidth() / 2 - MeasureText("WINNER PLAYER 2", 10) / 2, GetScreenHeight() / 2 - 20, 10, BLACK);
        }else{
            DrawText("NO WINNERS!", GetScreenWidth() / 2 - MeasureText("NO WINNERS!", 10) / 2, GetScreenHeight() / 2 - 20, 10, BLACK);
        }
        EndDrawing();
    }
    placar_final(&game);
    UnloadTexture(mario_frente);
    UnloadTexture(luigi_frente);
    UnloadTexture(mario_costas);
    UnloadTexture(luigi_costa);
    UnloadTexture(mario_direita);
    UnloadTexture(luigi_direita);
    UnloadTexture(mario_esquerda);
    UnloadTexture(luigi_esquerda);
    UnloadTexture(cenario1);
    UnloadTexture(cenario2);
    UnloadTexture(barreiras);
    UnloadTexture(barreiraquebravel);
    UnloadTexture(barreiralateral);
    UnloadTexture(tela_inicial);
    UnloadTexture(escolhemapa);
    UnloadTexture(tela_revanche);
    UnloadTexture(game_over);
    UnloadTexture(inimigo_1);
    UnloadTexture(inimigo_2);
    UnloadTexture(inimigo_3);
    UnloadTexture(inimigo_4);
    UnloadTexture(bomba);
    UnloadTexture(game_over);
    UnloadTexture(barreiracanto);
    UnloadTexture(tela_comeco);

    UnloadMusicStream(musicaInicio);
    UnloadMusicStream(musicaFim);
    UnloadMusicStream(musicaGame);
    UnloadMusicStream(sound_bomba);
    return 0;
}
//------------------------------------------------------------------------------------
// Module Functions Definitions (local)
//------------------------------------------------------------------------------------

// Initialize game variables
void InitGame(Game *g)
{

    framesCounter = 7200;
    g->num_maps = 2;
    g->hero1.pos = (Rectangle){40, 40, STD_SIZE_X, STD_SIZE_Y};
    g->hero1.color = BLACK;
    g->hero1.speed = 4;
    g->hero1.special = 0;
    g->gameover = 0;
    g->hero1.num_bombs = 1;
    g->hero1.put_bomb = 0;
    g->hero1.draw_bomb = 0;
    g->hero1.dead = 0;
    g->hero2.pos = (Rectangle){730, 530, STD_SIZE_X, STD_SIZE_Y};
    g->hero2.color = BLUE;
    g->hero2.speed = 4;
    g->hero2.special = 0;
    g->gameover = 0;
    g->hero2.num_bombs = 1;
    g->hero2.put_bomb = 0;
    g->hero2.draw_bomb = 0;
    g->hero2.dead = 0;
    g->enemies.pos = (Rectangle){400, 210, 20, 20};
    g->enemies.color = YELLOW;
    g->enemies.speed = 7;
    g->enemies2.pos=(Rectangle){400,370,20,20};
    g->enemies2.color=YELLOW;
    g->enemies2.speed=7;

    map0_setup(g);
    map1_setup(g);
    if(!revancheTop){
    nameScreen(g,&g->hero1,1);
    nameScreen(g,&g->hero2,2);
    }
    if(nomeEscolhido == 2){
        escolheMap(g);
    }

    for(int i=0;i<16;i++){
        g->maps[g->curr_map].blocks[i].isBroken= 0;
    }
    for(int i=0;i<3;i++){
        g->maps[g->curr_map].esp1[i].foiUsado = 0;
        g->maps[g->curr_map].esp2[i].foiUsado = 0;
        g->maps[g->curr_map].esp3[i].foiUsado = 0;
    }
}

// Update game (one frame)
void UpdateGame(Game *g)
{
    Map *m = &g->maps[g->curr_map];
    if (!g->gameover)
    {
        framesCounter--;

        if (framesCounter != 0.0)
        {
            update_hero1_pos(g);
            update_hero2_pos(g);
            update_bomb(g);
            update_enemy_pos(g);
            update_enemy2_pos(g);

            for (int i = 0; i < g->hero2.num_bombs; i++)
            {
                if(g->hero2.bombs[i].isActive==1){
                if(fabs(g->hero2.bombs[i].time - GetTime()) > 1 ){
                  if (CheckCollisionRecs(g->hero1.pos, g->hero2.bombs[i].explosion_down))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero2.bombs[i].explosion_left))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero2.bombs[i].explosion_right))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero2.bombs[i].explosion_up))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                }
                }
            }
            for (int i = 0; i < g->hero1.num_bombs; i++)
            {
                if(g->hero1.bombs[i].isActive==1){
                    if(fabs(g->hero1.bombs[i].time - GetTime()) > 1 ){
                    if (CheckCollisionRecs(g->hero2.pos, g->hero1.bombs[i].explosion_down))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero1.bombs[i].explosion_left))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero1.bombs[i].explosion_right))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero1.bombs[i].explosion_up))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    }
                }
            }
            for (int i = 0; i < g->hero1.num_bombs; i++)
            {
                if(g->hero1.bombs[i].isActive==1){
                    if(fabs(g->hero1.bombs[i].time - GetTime()) > 1 ){
                    if (CheckCollisionRecs(g->hero1.pos, g->hero1.bombs[i].explosion_down))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero1.bombs[i].explosion_left))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero1.bombs[i].explosion_right))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero1.bombs[i].explosion_up))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    }
                }
            }
            for (int i = 0; i < g->hero2.num_bombs; i++)
            {
                if(g->hero2.bombs[i].isActive==1){
                    if(fabs(g->hero2.bombs[i].time - GetTime()) > 1 ){
                    if (CheckCollisionRecs(g->hero2.pos, g->hero2.bombs[i].explosion_down))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero2.bombs[i].explosion_left))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero2.bombs[i].explosion_right))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero2.bombs[i].explosion_up))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    }
                }
            }
            if(g->hero2.dead == 1){
                g->hero2.derrotas++;
                g->hero1.vitorias++;
            }
            if(g->hero1.dead == 1){
                g->hero1.derrotas++;
                g->hero2.vitorias++;
            }

            int collidedBlockIndex1 = CheckCollisionWithBlocks(m,&g->hero1);
            if (collidedBlockIndex1 != -1) {
            // Bloco quebrável foi atingido
            m->blocks[collidedBlockIndex1].isBroken = 1;
            }
            int collidedBlockIndex2 = CheckCollisionWithBlocks(m,&g->hero2);
            if (collidedBlockIndex2 != -1) {
            // Bloco quebrável foi atingido
            m->blocks[collidedBlockIndex2].isBroken = 1;
            }

            //Especial 1
            int aux = PegouEspecial1(&g->hero1,g);
            if(aux != -1){
                g->maps[g->curr_map].esp1[aux].foiUsado = 1;
                g->hero1.speed+=2;
            }
            int aux2 = PegouEspecial1(&g->hero2,g);
            if(aux2 != -1){
                g->maps[g->curr_map].esp1[aux2].foiUsado = 1;
                g->hero2.speed+=2;
            }

            //Especial 2
            int auxi = PegouEspecial2(&g->hero1,g);
            if(auxi != -1){
                g->maps[g->curr_map].esp2[auxi].foiUsado = 1;
                g->hero1.num_bombs++;
               //g->hero1.bombs[g->hero1.num_bombs].isActive=0;
            }
            int auxi2 = PegouEspecial2(&g->hero2,g);
            if(auxi2 != -1){
                g->maps[g->curr_map].esp2[auxi2].foiUsado = 1;
                g->hero2.num_bombs++;
                //g->hero2.bombs[g->hero2.num_bombs].isActive= 0;
            }

            //Especial 3
            int auxil = PegouEspecial3(&g->hero1,g);
            if(auxil != -1){
                g->maps[g->curr_map].esp3[auxil].foiUsado = 1;
                for(int i=0;i<g->hero1.num_bombs;i++){
                    g->hero1.bombs[i].distance +=2;
                }
            }
            int auxil2 = PegouEspecial3(&g->hero2,g);
            if(auxil2 != -1){
                g->maps[g->curr_map].esp3[auxil2].foiUsado = 1;
                for(int i=0;i<g->hero2.num_bombs;i++){
                    g->hero2.bombs[i].distance +=2;
                }
            }
        }
    }
}

void UpdatePosGame(Game *g){
    if(framesCounter==0.0){
        update_bomb(g);
        for (int i = 0; i < g->hero2.num_bombs; i++){
            if(g->hero2.bombs[i].isActive==1){
                if(fabs(g->hero2.bombs[i].time - GetTime()) > 1 ){
                  if (CheckCollisionRecs(g->hero1.pos, g->hero2.bombs[i].explosion_down))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero2.bombs[i].explosion_left))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero2.bombs[i].explosion_right))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero2.bombs[i].explosion_up))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                }
            }
        }
        for (int i = 0; i < g->hero1.num_bombs; i++){
            if(g->hero1.bombs[i].isActive==1){
                if(fabs(g->hero1.bombs[i].time - GetTime()) > 1 ){
                    if (CheckCollisionRecs(g->hero2.pos, g->hero1.bombs[i].explosion_down))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero1.bombs[i].explosion_left))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero1.bombs[i].explosion_right))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero1.bombs[i].explosion_up))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                }
            }
        }
        for (int i = 0; i < g->hero1.num_bombs; i++){
            if(g->hero1.bombs[i].isActive==1){
                if(fabs(g->hero1.bombs[i].time - GetTime()) > 1 ){
                    if (CheckCollisionRecs(g->hero1.pos, g->hero1.bombs[i].explosion_down))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero1.bombs[i].explosion_left))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero1.bombs[i].explosion_right))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero1.pos, g->hero1.bombs[i].explosion_up))
                    {
                        g->gameover = 1;
                        g->hero1.dead = 1;
                    }
                }
            }
        }
        for (int i = 0; i < g->hero2.num_bombs; i++){
            if(g->hero2.bombs[i].isActive==1){
                if(fabs(g->hero2.bombs[i].time - GetTime()) > 1 ){
                    if (CheckCollisionRecs(g->hero2.pos, g->hero2.bombs[i].explosion_down))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero2.bombs[i].explosion_left))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero2.bombs[i].explosion_right))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                    if (CheckCollisionRecs(g->hero2.pos, g->hero2.bombs[i].explosion_up))
                    {
                        g->gameover = 1;
                        g->hero2.dead = 1;
                    }
                }
            }
        }
        if(g->hero2.dead == 1){
                g->hero2.derrotas++;
                g->hero1.vitorias++;
        }
        if(g->hero1.dead == 1){
                g->hero1.derrotas++;
                g->hero2.vitorias++;
        }
    }
    if(fabs( 12 - GetTime()) > 8 && !g->gameover){
        g->gameover=1;
    }
}

// Draw game (one frame)
void DrawGame(Game *g)
{
    BeginDrawing();
    ClearBackground(RAYWHITE);
    if (!g->gameover){

        if (g->map == 1) {
            Map *map = &g->maps[g->curr_map];
            PlayMusicStream(musicaGame);
            UpdateMusicStream(musicaGame);
            DrawTexture(cenario1, 0, 0, WHITE);
            draw_borders(g);
            DrawSpecial(g);
            DrawBlocks(g);
            draw_map(g);
            draw_bomb(g);
                
    }
        if (g->map == 2) {
            Map *map = &g->maps[g->curr_map];
            PlayMusicStream(musicaGame);
            UpdateMusicStream(musicaGame);
            DrawTexture(cenario2, 0, 0, WHITE);
            draw_borders(g);
            DrawSpecial(g);
            DrawBlocks(g);
            draw_map(g);
            draw_bomb(g);       
    }
        DrawTexture(inimigo_1,g->enemies.pos.x,g->enemies.pos.y, WHITE);
        DrawTexture(inimigo_2,g->enemies2.pos.x,g->enemies2.pos.y, WHITE);
        if (g->enemies.direction == KEY_RIGHT){
            DrawTexture(inimigo_3,g->enemies.pos.x,g->enemies.pos.y, WHITE);
        }
        else if (g->enemies2.direction == KEY_RIGHT){
            DrawTexture(inimigo_4,g->enemies2.pos.x,g->enemies2.pos.y, WHITE);
        }
        else if (g->enemies.direction == KEY_LEFT){
            DrawTexture(inimigo_1,g->enemies.pos.x,g->enemies.pos.y, WHITE);
        }
        else if (g->enemies2.direction == KEY_LEFT){
            DrawTexture(inimigo_2,g->enemies2.pos.x,g->enemies2.pos.y, WHITE);
        }
        DrawTexture(mario_frente, g->hero1.pos.x, g->hero1.pos.y, WHITE);
        DrawTexture(luigi_frente, g->hero2.pos.x, g->hero2.pos.y, WHITE);
        if (IsKeyDown(KEY_A)){
                DrawTexture(mario_esquerda, g->hero1.pos.x, g->hero1.pos.y, WHITE);
            }
        else if(IsKeyDown(KEY_W)){
                DrawTexture(mario_costas, g->hero1.pos.x, g->hero1.pos.y, WHITE);
            }
        else if (IsKeyDown(KEY_D)){
                DrawTexture(mario_direita, g->hero1.pos.x, g->hero1.pos.y, WHITE);
            }
        else if (IsKeyDown(KEY_S)){
                DrawTexture(mario_frente, g->hero1.pos.x, g->hero1.pos.y, WHITE);
            }
        else if (IsKeyDown(KEY_LEFT)){
                DrawTexture(luigi_esquerda, g->hero2.pos.x, g->hero2.pos.y, WHITE);
            }
        else if(IsKeyDown(KEY_UP)){
                DrawTexture(luigi_costa, g->hero2.pos.x, g->hero2.pos.y, WHITE);
            }
        else if (IsKeyDown(KEY_RIGHT)){
                DrawTexture(luigi_direita, g->hero2.pos.x, g->hero2.pos.y, WHITE);
            }
        else if (IsKeyDown(KEY_DOWN)){
                DrawTexture(luigi_frente, g->hero2.pos.x, g->hero2.pos.y, WHITE);
            }
        if (framesCounter > 0)
        {
            DrawText(TextFormat("%d : %d",g->hero1.vitorias,g->hero2.vitorias),380,10,25,WHITE);
            DrawText(TextFormat("TIME: %.02f SECONDS", (float)framesCounter / 60), 10, 10, 20, WHITE);
        }
        else
        {
            DrawText(TextFormat("%d : %d",g->hero1.vitorias,g->hero2.vitorias),380,10,25,WHITE);
            DrawText(TextFormat("TIME:00.00 SECONDS", (float)framesCounter / 60), 10, 10, 20, WHITE);
        }
    }
    EndDrawing();
}

// Update and Draw (one frame)
void UpdateDrawFrame(Game *g)
{
    if (framesCounter != 0)
    {
        UpdateGame(g);
        DrawGame(g);
    }
    else
    {
        
        UpdatePosGame(g);
        DrawGame(g);
    
    }
}

void TelaInicial(Game *g){
    g->res = 0;
    while(!IsKeyPressed(KEY_ENTER)){
        ClearBackground(RAYWHITE);
        BeginDrawing();
        PlayMusicStream(musicaInicio);
        UpdateMusicStream(musicaInicio);
        DrawTexture(tela_comeco, 0, 0, WHITE);
        EndDrawing();
    }
    if(IsKeyPressed(KEY_ENTER)){
        g->res = 1;
    }
}

void escolheMap(Game *g)
{
    int selectedMap = 0; // 0 para nenhum mapa selecionado
    while (!IsKeyPressed(KEY_BACKSPACE) || revancheTop == 1)
    {
        BeginDrawing();
        ClearBackground(RAYWHITE);
        PlayMusicStream(musicaInicio);
        UpdateMusicStream(musicaInicio);
        DrawTexture(escolhemapa, 0, 0, WHITE);

        DrawText("CHOOSE MAP (PRESS BACKSPACE)", GetScreenWidth() / 2 - MeasureText("CHOOSE MAP (PRESS BACKSPACE)", 20) / 2, GetScreenHeight() / 2 - 50, 20, BLACK);

        DrawText("MAP 1", GetScreenWidth() / 2 - MeasureText("MAP 1", 20) / 2 + 10, GetScreenHeight() / 2 - 20, (selectedMap == 1) ? 20 : 10, (selectedMap == 1) ? BLACK : GRAY);
        DrawText("MAP 2", GetScreenWidth() / 2 - MeasureText("MAP 2", 20) / 2 + 10, GetScreenHeight() / 2, (selectedMap == 2) ? 20 : 10, (selectedMap == 2) ? BLACK : GRAY);

        EndDrawing();

        if (IsKeyPressed(KEY_UP))
        {
            selectedMap = 1;
        }
        else if (IsKeyPressed(KEY_DOWN))
        {
            selectedMap = 2;
        }
        if(IsKeyPressed(KEY_BACKSPACE)){
          revancheTop = 0;
        }
    }

    if (selectedMap != 0)
    {
        g->map = selectedMap;
        g->curr_map = selectedMap - 1; // Ajuste para índices de array (começando de 0)
    }
    return;
}

int revanche(Game *g)
{
    int simOUnao = 0; // 0 para nenhum mapa selecionado

    while (!IsKeyPressed(KEY_BACKSPACE))
    {
        BeginDrawing();
        ClearBackground(RAYWHITE);
        PlayMusicStream(musicaFim);
        UpdateMusicStream(musicaFim);
        DrawTexture(tela_revanche, 0, 0 , WHITE);
        if (g->hero2.dead == 1)
        {
            DrawText(TextFormat("WINNER PLAYER 1: %s!",g->hero1.name), GetScreenWidth() / 2 - MeasureText("WINNNER PLAYER 1!", 50) / 2, GetScreenHeight() / 2 - 120, 50, BLACK);
        }
        else if (g->hero1.dead == 1)
        {
            DrawText(TextFormat("WINNER PLAYER 2: %s!",g->hero2.name), GetScreenWidth() / 2 - MeasureText("WINNER PLAYER 2!", 50) / 2, GetScreenHeight() / 2 - 120, 50, BLACK);
        }else{
            DrawText("NO WINNERS!", GetScreenWidth() / 2 - MeasureText("NO WINNERS!", 50) / 2, GetScreenHeight() / 2 - 120, 50, BLACK);
        }
        DrawText(TextFormat("%d : %d",g->hero1.vitorias,g->hero2.vitorias),GetScreenWidth()/2 - MeasureText("0 : 0",25)/2,GetScreenHeight()/2 - 80,25,BLACK);
        DrawText("REVANCHE (PRESS BACKSPACE)", GetScreenWidth() / 2 - MeasureText("REVANCHE (PRESS BACKSPACE)", 20) / 2, GetScreenHeight() / 2 - 50, 20, BLACK);

        DrawText("SIM", GetScreenWidth() / 2 - MeasureText("SIM", 25) / 2 + 10, GetScreenHeight() / 2 - 10, (simOUnao == 1) ? 25 : 15, (simOUnao == 1) ? BLACK : GRAY);
        DrawText("NÃO", GetScreenWidth() / 2 - MeasureText("NÃO", 25) / 2 + 10, GetScreenHeight() / 2 + 20, (simOUnao == 2) ? 25 : 15, (simOUnao == 2) ? BLACK : GRAY);

        EndDrawing();

        if (IsKeyPressed(KEY_UP))
        {
            simOUnao = 1;
        }
        else if (IsKeyPressed(KEY_DOWN))
        {
            simOUnao = 2;
        }
    }

    return simOUnao;
}

void nameScreen(Game *g,Hero *h,int n){
    char text[50];
    sprintf(text, "Enter Player %d Name: ",n);
    char playerName[100] = {0};
    int cursor = 0;

    while(!IsKeyPressed(KEY_ENTER)|| nomeEscolhido == 0 || nomeEscolhido == 1){
        BeginDrawing();
        ClearBackground(RAYWHITE);
        PlayMusicStream(musicaInicio);
        UpdateMusicStream(musicaInicio);
        DrawTexture(tela_inicial, 0, 0, WHITE);
        DrawText(text, GetScreenWidth()/2 - MeasureText(text,20)/2,50,20,BLACK);
        
        DrawText(TextSubtext(playerName,0,cursor),GetScreenWidth()/2 - MeasureText(playerName,20)/2,100,20,BLACK);
        
        EndDrawing();

        int key = GetKeyPressed();
        if(key !=0){
            if(key==KEY_BACKSPACE && cursor >0){
                playerName[--cursor]='\0';
            }else if(key>=32 && key<=125 && cursor <100-1){
                playerName[cursor++]=(char)key;
            }
        }
        if(IsKeyPressed(KEY_ENTER)){
            nomeEscolhido++;
            break;
        }
    }
    strcpy(h->name,playerName);
    for(int i=0;i<100;i++){
        playerName[i]='\0';
    }
}

void draw_borders(Game *g)
{
    DrawTexture(barreiralateral,SCREEN_BORDER, g->screenHeight, WHITE);
    DrawTexture(barreiracanto, g->screenWidth, SCREEN_BORDER, WHITE);
    DrawTexture(barreiralateral,g->screenWidth, g->screenHeight, WHITE);
    DrawTexture(barreiracanto, SCREEN_BORDER, g->screenHeight, WHITE);
}

void DrawBlocks(Game *g) {
    if(g->map==1){
    for (int i = 0; i <16; i++) {
        if (g->maps[0].blocks[i].isBroken==0) {
            if(g->maps[0].blocks[i].rect.x !=0 && g->maps[0].blocks[i].rect.y !=0){
            DrawTexture(barreiraquebravel, g->maps[0].blocks[i].rect.x, g->maps[0].blocks[i].rect.y, WHITE);
            }
        }
    }
    }

    if(g->map==2){
        for (int i = 0; i < 17; i++) {
        if (g->maps[1].blocks[i].isBroken==0) {
            if(g->maps[1].blocks[i].rect.x !=0 && g->maps[1].blocks[i].rect.y !=0){
            DrawTexture(barreiraquebravel, g->maps[1].blocks[i].rect.x, g->maps[1].blocks[i].rect.y, WHITE);
            }
        }

    }
    }
}

void DrawSpecial(Game *g){
    if(g->map==1){
        for(int i=0;i<3;i++){
            if(g->maps[0].esp1[i].foiUsado==0){
            DrawTexture(velocidade,g->maps[0].esp1[i].spc.x, g->maps[0].esp1[i].spc.y, WHITE);
            }
        }
        for(int i=0;i<3;i++){
            if(g->maps[0].esp2[i].foiUsado==0){
            DrawTexture(bomba,g->maps[0].esp2[i].spc.x, g->maps[0].esp2[i].spc.y, WHITE);
            }
        }
        for(int i=0;i<3;i++){
            if(g->maps[0].esp3[i].foiUsado==0){
            DrawTexture(raio,g->maps[0].esp3[i].spc.x, g->maps[0].esp3[i].spc.y, WHITE);
            }
        }
    }

    if(g->map==2){
        for(int i=0;i<3;i++){
            if(g->maps[1].esp1[i].foiUsado==0){
                DrawTexture(velocidade,g->maps[1].esp1[i].spc.x, g->maps[1].esp1[i].spc.y, WHITE);
            }
        }
        for(int i=0;i<3;i++){
            if(g->maps[1].esp2[i].foiUsado==0){
            DrawTexture(bomba,g->maps[1].esp2[i].spc.x, g->maps[1].esp2[i].spc.y, WHITE);
            }
        }
        for(int i=0;i<3;i++){
            if(g->maps[1].esp3[i].foiUsado==0){
            DrawTexture(raio,g->maps[1].esp3[i].spc.x, g->maps[1].esp3[i].spc.y, WHITE);
            }
        }
    }
}


void draw_map(Game *g)
{
    if (g->map == 1)
    {
        Map *map = &g->maps[g->curr_map];
        for(int i = 0; i<36; i++){
            DrawTexture(barreiras_1, map->barriers[i].x,map->barriers[i].y, WHITE);    
              
    }

    }
    
    if (g->map == 2)
    {
        Map *map = &g->maps[g->curr_map];
        for (int i = 0; i < 36; i++)
        {
            DrawTexture(barreiras, map->barriers[i].x,map->barriers[i].y, WHITE);  
        }
    }

}

void draw_bomb(Game *g)
{
    // Bomba do Jogador 1
    for (int i = 0; i < g->hero1.num_bombs; i++)
    {

            
        if (g->hero1.bombs[i].isActive == 1)
        {
            if(fabs(g->hero1.bombs[i].time-GetTime())>1){
            DrawRectangleRec(g->hero1.bombs[i].explosion_right, RED);
            DrawRectangleRec(g->hero1.bombs[i].explosion_left, RED);
            DrawRectangleRec(g->hero1.bombs[i].explosion_up, RED);
            DrawRectangleRec(g->hero1.bombs[i].explosion_down, RED);
            UpdateMusicStream(sound_bomba);
        }
        else{
            DrawTexture(bomba, g->hero1.bombs[i].pos.x, g->hero1.bombs[i].pos.y, WHITE);
         }
        } 
    }

    // Bomba do Jogador 2
    for (int j = 0; j < g->hero2.num_bombs; j++)
    {
        if (g->hero2.bombs[j].isActive == 1)
        {
            if(fabs(g->hero2.bombs[j].time-GetTime())>1){
            DrawRectangleRec(g->hero2.bombs[j].pos, RED);
            DrawRectangleRec(g->hero2.bombs[j].explosion_right, RED);
            DrawRectangleRec(g->hero2.bombs[j].explosion_left, RED);
            DrawRectangleRec(g->hero2.bombs[j].explosion_up, RED);
            DrawRectangleRec(g->hero2.bombs[j].explosion_down, RED);
            UpdateMusicStream(sound_bomba);
        }
        else{
            DrawTexture(bomba, g->hero2.bombs[j].pos.x, g->hero2.bombs[j].pos.y, WHITE);
        }
        }
    }
}

void update_hero1_pos(Game *g)
{

    Hero *h = &g->hero1;
    Map *m = &g->maps[g->curr_map];
    Enemy *e = &g->enemies;
    Hero *h2 = &g->hero2;
    Enemy *e2=&g->enemies2;

    // Movimenta o Jogador 1
    if (framesCounter != 0.0)
    { // Deixa o jogador se movimentar enquanto o cronometro não zerar
        if (IsKeyDown(KEY_A))
        {
            if (h->pos.x > SCREEN_BORDER)
                h->pos.x -= h->speed;
            if (barrier_collision(m, h->pos) || bomb_collision(h, h->pos) || bomb_collision(h2, h->pos) || hero_collision(h->pos,h2->pos))
                h->pos.x += h->speed;
        }
        else if (IsKeyDown(KEY_D))
        {
            if (h->pos.x + h->pos.width < g->screenWidth - SCREEN_BORDER)
                h->pos.x += h->speed;
            if (barrier_collision(m, h->pos) || bomb_collision(h, h->pos) || bomb_collision(h2, h->pos) || hero_collision(h->pos,h2->pos))
                h->pos.x -= h->speed;
        }
        else if (IsKeyDown(KEY_W))
        {
            if (h->pos.y > SCREEN_BORDER)
                h->pos.y -= h->speed;
            if (barrier_collision(m, h->pos) || bomb_collision(h, h->pos) || bomb_collision(h2, h->pos)|| hero_collision(h->pos,h2->pos))
                h->pos.y += h->speed;
        }
        else if (IsKeyDown(KEY_S))
        {
            if (h->pos.y + h->pos.height < g->screenHeight - SCREEN_BORDER)
                h->pos.y += h->speed;
            if (barrier_collision(m, h->pos) || bomb_collision(h, h->pos) || bomb_collision(h2, h->pos)|| hero_collision(h->pos,h2->pos))
                h->pos.y -= h->speed;
        }

        if (CheckCollisionRecs(e->pos, h->pos))
        {
            h->pos = (Rectangle){40, 40, STD_SIZE_X, STD_SIZE_Y};
        }
        if (CheckCollisionRecs(e2->pos, h->pos))
        {
            h->pos = (Rectangle){40, 40, STD_SIZE_X, STD_SIZE_Y};
        }
    }
}

void update_hero2_pos(Game *g)
{

    Map *m = &g->maps[g->curr_map];
    Hero *h2 = &g->hero2;
    Enemy *e = &g->enemies;
    Enemy *e2=&g->enemies2;
    Hero *h = &g->hero1;

    // Movimenta o Jogador 2
    if (framesCounter != 0.0)
    {
        if (IsKeyDown(KEY_LEFT))
        {
            if (h2->pos.x > SCREEN_BORDER)
                h2->pos.x -= h2->speed;
            if (barrier_collision(m, h2->pos) || bomb_collision(h2, h2->pos) || bomb_collision(h, h2->pos)|| hero_collision(h->pos,h2->pos))
                h2->pos.x += h2->speed;
        }
        else if (IsKeyDown(KEY_RIGHT))
        {
            if (h2->pos.x + h2->pos.width < g->screenWidth - SCREEN_BORDER)
                h2->pos.x += h2->speed;
            if (barrier_collision(m, h2->pos) || bomb_collision(h2, h2->pos) || bomb_collision(h, h2->pos)|| hero_collision(h->pos,h2->pos))
                h2->pos.x -= h2->speed;
        }
        else if (IsKeyDown(KEY_UP))
        {
            if (h2->pos.y > SCREEN_BORDER)
                h2->pos.y -= h2->speed;
            if (barrier_collision(m, h2->pos) || bomb_collision(h2, h2->pos) || bomb_collision(h, h2->pos)|| hero_collision(h->pos,h2->pos))
                h2->pos.y += h2->speed;
        }
        else if (IsKeyDown(KEY_DOWN))
        {
            if (h2->pos.y + h2->pos.height < g->screenHeight - SCREEN_BORDER)
                h2->pos.y += h2->speed;
            if (barrier_collision(m, h2->pos) || bomb_collision(h2, h2->pos) || bomb_collision(h, h2->pos)|| hero_collision(h->pos,h2->pos))
                h2->pos.y -= h2->speed;
        }

        if (CheckCollisionRecs(e->pos, h2->pos))
        {
            h2->pos = (Rectangle){730, 530, STD_SIZE_X, STD_SIZE_Y};
        }
        if (CheckCollisionRecs(e2->pos, h2->pos))
        {
            h2->pos = (Rectangle){730, 530, STD_SIZE_X, STD_SIZE_Y};
        }
    }
}

void update_enemy_pos(Game *g)
{
    Map *m = &g->maps[g->curr_map];

    if (g->enemies.direction == KEY_LEFT)
    {
        if (g->enemies.pos.x > SCREEN_BORDER)
            g->enemies.pos.x -= g->enemies.speed;
        else
        {
            g->enemies.direction = KEY_RIGHT + (rand() % 4);
        }
        if (barrier_collision(m, g->enemies.pos))
        {
            g->enemies.pos.x += g->enemies.speed;
            g->enemies.direction = KEY_RIGHT + (rand() % 4);
        }
    }
    else if (g->enemies.direction == KEY_RIGHT)
    {
        if (g->enemies.pos.x + g->enemies.pos.width < g->screenWidth - SCREEN_BORDER)
            g->enemies.pos.x += g->enemies.speed;
        else
        {
            g->enemies.direction = KEY_RIGHT + (rand() % 4);
        }
        if (barrier_collision(m, g->enemies.pos))
        {
            g->enemies.pos.x -= g->enemies.speed;
            g->enemies.direction = KEY_RIGHT + (rand() % 4);
        }
    }
    else if (g->enemies.direction == KEY_UP)
    {
        if (g->enemies.pos.y > SCREEN_BORDER)
            g->enemies.pos.y -= g->enemies.speed;
        else
        {
            g->enemies.direction = KEY_RIGHT + (rand() % 4);
        }
        if (barrier_collision(m, g->enemies.pos))
        {
            g->enemies.pos.y += g->enemies.speed;
            g->enemies.direction = KEY_RIGHT + (rand() % 4);
        }
    }
    else if (g->enemies.direction == KEY_DOWN)
    {
        if (g->enemies.pos.y + g->enemies.pos.height < g->screenHeight - SCREEN_BORDER)
            g->enemies.pos.y += g->enemies.speed;
        else
        {
            g->enemies.direction = KEY_RIGHT + (rand() % 4);
        }
        if (barrier_collision(m, g->enemies.pos))
        {
            g->enemies.pos.y -= g->enemies.speed;
            g->enemies.direction = KEY_RIGHT + (rand() % 4);
        }
    }
    if (rand() % 100 == 1) // 1% de chance do inimigo mudar de direcao
        g->enemies.direction = KEY_RIGHT + (rand() % 4);
}

void update_enemy2_pos(Game *g){
    Map *m = &g->maps[g->curr_map];

    if (g->enemies2.direction == KEY_LEFT)
    {
        if (g->enemies2.pos.x > SCREEN_BORDER)
            g->enemies2.pos.x -= g->enemies2.speed;
        else
        {
            g->enemies2.direction = KEY_RIGHT + (rand() % 4);
        }
        if (barrier_collision(m, g->enemies2.pos))
        {
            g->enemies2.pos.x += g->enemies2.speed;
            g->enemies2.direction = KEY_RIGHT + (rand() % 4);
        }
    }
    else if (g->enemies2.direction == KEY_RIGHT)
    {
        if (g->enemies2.pos.x + g->enemies2.pos.width < g->screenWidth - SCREEN_BORDER)
            g->enemies2.pos.x += g->enemies2.speed;
        else
        {
            g->enemies2.direction = KEY_RIGHT + (rand() % 4);
        }
        if (barrier_collision(m, g->enemies2.pos))
        {
            g->enemies2.pos.x -= g->enemies2.speed;
            g->enemies2.direction = KEY_RIGHT + (rand() % 4);
        }
    }
    else if (g->enemies2.direction == KEY_UP)
    {
        if (g->enemies2.pos.y > SCREEN_BORDER)
            g->enemies2.pos.y -= g->enemies2.speed;
        else
        {
            g->enemies2.direction = KEY_RIGHT + (rand() % 4);
        }
        if (barrier_collision(m, g->enemies2.pos))
        {
            g->enemies2.pos.y += g->enemies2.speed;
            g->enemies2.direction = KEY_RIGHT + (rand() % 4);
        }
    }
    else if (g->enemies2.direction == KEY_DOWN)
    {
        if (g->enemies2.pos.y + g->enemies2.pos.height < g->screenHeight - SCREEN_BORDER)
            g->enemies2.pos.y += g->enemies2.speed;
        else
        {
            g->enemies2.direction = KEY_RIGHT + (rand() % 4);
        }
        if (barrier_collision(m, g->enemies2.pos))
        {
            g->enemies2.pos.y -= g->enemies2.speed;
            g->enemies2.direction = KEY_RIGHT + (rand() % 4);
        }
    }
    if (rand() % 100 == 1) // 1% de chance do inimigo mudar de direcao
        g->enemies2.direction = KEY_RIGHT + (rand() % 4);

}

void update_bomb(Game *g)
{
        Map *actual_map = &g->maps[g->curr_map];
        // Libera a Bomba do Jogador 1
        if (IsKeyPressed(KEY_SPACE))
        {
            g->hero1.put_bomb = 1;
        }

        if (g->hero1.put_bomb == 1)
        {
            for (int i = 0; i < g->hero1.num_bombs; i++)
            {
                if (g->hero1.bombs[i].isActive == 0)
                {
                    g->hero1.bombs[i].isActive = 1;
                    g->hero1.bombs[i].pos = (Rectangle){g->hero1.pos.x, g->hero1.pos.y, STD_SIZE_BOMB_X, STD_SIZE_BOMB_Y};
                    g->hero1.bombs[i].explosion_right = (Rectangle){g->hero1.pos.x, g->hero1.pos.y, STD_SIZE_BOMB_X, STD_SIZE_BOMB_Y};
                    g->hero1.bombs[i].explosion_left = (Rectangle){g->hero1.pos.x, g->hero1.pos.y, STD_SIZE_BOMB_X, STD_SIZE_BOMB_Y};
                    g->hero1.bombs[i].explosion_down = (Rectangle){g->hero1.pos.x, g->hero1.pos.y, STD_SIZE_BOMB_X, STD_SIZE_BOMB_Y};
                    g->hero1.bombs[i].explosion_up = (Rectangle){g->hero1.pos.x, g->hero1.pos.y, STD_SIZE_BOMB_X, STD_SIZE_BOMB_Y};
                    g->hero1.bombs[i].time = GetTime();
                    g->hero1.bombs[i].local = 1;
                    break;
                }
                
            }
        }
        for (int i = 0; i < g->hero1.num_bombs; i++)
        {
            if(g->hero1.bombs[i].isActive && fabs(g->hero1.bombs[i].time - GetTime()) < 1 && bombs_hipercollision(&g->hero1,&(g->hero1.bombs[i].pos))==1){
                g->hero1.bombs[i].time=GetTime()-1;
            }
            if(g->hero2.bombs[i].isActive && fabs(g->hero2.bombs[i].time - GetTime()) < 1 && bombs_hipercollision(&g->hero1,&(g->hero2.bombs[i].pos))==1){
                g->hero2.bombs[i].time=GetTime()-1;
            }
            if (g->hero1.bombs[i].isActive == 1)
            {
                if (!CheckCollisionRecs(g->hero1.bombs[i].pos, g->hero1.pos))
                    g->hero1.bombs[i].local = 0;
                if (fabs(g->hero1.bombs[i].time - GetTime()) > 1 && fabs(g->hero1.bombs[i].time - GetTime()) < 3)
                {
                    Rectangle verify_bomb;
                    Rectangle rectangle_bomb;
                    int grow_tax = g->hero1.bombs[i].distance;
                    if (g->hero1.bombs[i].explosion_right.width < g->hero1.bombs[i].distance * STD_SIZE_X)
                    {
                        rectangle_bomb = g->hero1.bombs[i].explosion_right;
                        Rectangle verify_bomb = (Rectangle){rectangle_bomb.x, rectangle_bomb.y, rectangle_bomb.width + grow_tax, rectangle_bomb.height};
                        if (!barrier2_collision(actual_map, verify_bomb))
                        {
                            g->hero1.bombs[i].explosion_right.width += grow_tax;
                        }
                    }

                    if (g->hero1.bombs[i].explosion_left.width < g->hero1.bombs[i].distance * STD_SIZE_X)
                    {
                        rectangle_bomb = g->hero1.bombs[i].explosion_left;
                        Rectangle verify_bomb = (Rectangle){rectangle_bomb.x - grow_tax, rectangle_bomb.y, rectangle_bomb.width + grow_tax, rectangle_bomb.height};
                        if (!barrier2_collision(actual_map, verify_bomb))
                        {
                            g->hero1.bombs[i].explosion_left.width += grow_tax;
                            g->hero1.bombs[i].explosion_left.x -= grow_tax;
                        }
                    }

                    if (g->hero1.bombs[i].explosion_up.height < g->hero1.bombs[i].distance * STD_SIZE_Y)
                    {
                        rectangle_bomb = g->hero1.bombs[i].explosion_up;
                        Rectangle verify_bomb = (Rectangle){rectangle_bomb.x, rectangle_bomb.y, rectangle_bomb.width, rectangle_bomb.height + grow_tax};
                        if (!barrier2_collision(actual_map, verify_bomb))
                        {
                            g->hero1.bombs[i].explosion_up.height += grow_tax;
                        }
                    }

                    if (g->hero1.bombs[i].explosion_down.height < g->hero1.bombs[i].distance * STD_SIZE_Y)
                    {
                        rectangle_bomb = g->hero1.bombs[i].explosion_down;
                        Rectangle verify_bomb = (Rectangle){rectangle_bomb.x, rectangle_bomb.y - grow_tax, rectangle_bomb.width, rectangle_bomb.height + grow_tax};
                        if (!barrier2_collision(actual_map, verify_bomb))
                        {
                            g->hero1.bombs[i].explosion_down.height += grow_tax;
                            g->hero1.bombs[i].explosion_down.y -= grow_tax;
                        }
                    }
                }
                else if (fabs(g->hero1.bombs[i].time - GetTime()) > 1)
                {
                    g->hero1.bombs[i].isActive = 0;
                    g->hero1.bombs[i].local = 1;
                }
            }
        }
    

        g->hero1.put_bomb = 0;
    
    // Libera a Bomba do Jogador 2
        if (IsKeyPressed(KEY_ENTER))
        {
            g->hero2.put_bomb = 1;
        }

        if (g->hero2.put_bomb == 1)
        {
            for (int j = 0; j < g->hero2.num_bombs; j++)
            {
                if (g->hero2.bombs[j].isActive == 0)
                {
                    g->hero2.bombs[j].isActive = 1;
                    g->hero2.bombs[j].pos = (Rectangle){g->hero2.pos.x, g->hero2.pos.y, STD_SIZE_BOMB_X, STD_SIZE_BOMB_Y};
                    g->hero2.bombs[j].explosion_right = (Rectangle){g->hero2.pos.x, g->hero2.pos.y, STD_SIZE_BOMB_X, STD_SIZE_BOMB_Y};
                    g->hero2.bombs[j].explosion_left = (Rectangle){g->hero2.pos.x, g->hero2.pos.y, STD_SIZE_BOMB_X, STD_SIZE_BOMB_Y};
                    g->hero2.bombs[j].explosion_down = (Rectangle){g->hero2.pos.x, g->hero2.pos.y, STD_SIZE_BOMB_X, STD_SIZE_BOMB_Y};
                    g->hero2.bombs[j].explosion_up = (Rectangle){g->hero2.pos.x, g->hero2.pos.y, STD_SIZE_BOMB_X, STD_SIZE_BOMB_Y};

                    g->hero2.bombs[j].time = GetTime();
                    g->hero2.bombs[j].local = 1;
                    break;
                }
            }
        }
        for (int j = 0; j < g->hero2.num_bombs; j++)
        {
            if(g->hero2.bombs[j].isActive && fabs(g->hero2.bombs[j].time - GetTime()) < 1 && bombs_hipercollision(&g->hero2,&(g->hero2.bombs[j].pos))==1){
                g->hero2.bombs[j].time=GetTime()-1;
            }
            if(g->hero1.bombs[j].isActive && fabs(g->hero1.bombs[j].time - GetTime()) < 1 && bombs_hipercollision(&g->hero2,&(g->hero1.bombs[j].pos))==1){
                g->hero1.bombs[j].time=GetTime()-1;
            }
            if (g->hero2.bombs[j].isActive == 1)
            {
                if (!CheckCollisionRecs(g->hero2.bombs[j].pos, g->hero2.pos))
                    g->hero2.bombs[j].local = 0;
                if (fabs(g->hero2.bombs[j].time - GetTime()) > 1 && fabs(g->hero2.bombs[j].time - GetTime()) < 3)
                {
                    Rectangle verify_bomb;
                    Rectangle rectangle_bomb;
                    int grow_tax = g->hero2.bombs[j].distance;
                    if (g->hero2.bombs[j].explosion_right.width < g->hero2.bombs[j].distance * STD_SIZE_X)
                    {
                        rectangle_bomb = g->hero2.bombs[j].explosion_right;
                        Rectangle verify_bomb = (Rectangle){rectangle_bomb.x, rectangle_bomb.y, rectangle_bomb.width + grow_tax, rectangle_bomb.height};
                        if (!barrier2_collision(actual_map, verify_bomb))
                        {
                            g->hero2.bombs[j].explosion_right.width += grow_tax;
                        }
                    }

                    if (g->hero2.bombs[j].explosion_left.width < g->hero2.bombs[j].distance * STD_SIZE_X)
                    {
                        rectangle_bomb = g->hero2.bombs[j].explosion_left;
                        Rectangle verify_bomb = (Rectangle){rectangle_bomb.x - grow_tax, rectangle_bomb.y, rectangle_bomb.width + grow_tax, rectangle_bomb.height};
                        if (!barrier2_collision(actual_map, verify_bomb))
                        {
                            g->hero2.bombs[j].explosion_left.width += grow_tax;
                            g->hero2.bombs[j].explosion_left.x -= grow_tax;
                        }
                    }

                    if (g->hero2.bombs[j].explosion_up.height < g->hero2.bombs[j].distance * STD_SIZE_Y)
                    {
                        rectangle_bomb = g->hero2.bombs[j].explosion_up;
                        Rectangle verify_bomb = (Rectangle){rectangle_bomb.x, rectangle_bomb.y, rectangle_bomb.width, rectangle_bomb.height + grow_tax};
                        if (!barrier2_collision(actual_map, verify_bomb))
                        {
                            g->hero2.bombs[j].explosion_up.height += grow_tax;
                        }
                    }

                    if (g->hero2.bombs[j].explosion_down.height < g->hero2.bombs[j].distance * STD_SIZE_Y)
                    {
                        rectangle_bomb = g->hero2.bombs[j].explosion_down;
                        Rectangle verify_bomb = (Rectangle){rectangle_bomb.x, rectangle_bomb.y - grow_tax, rectangle_bomb.width, rectangle_bomb.height + grow_tax};
                        if (!barrier2_collision(actual_map, verify_bomb))
                        {
                            g->hero2.bombs[j].explosion_down.height += grow_tax;
                            g->hero2.bombs[j].explosion_down.y -= grow_tax;
                        }
                    }
                }
                else if (fabs(g->hero2.bombs[j].time - GetTime()) > 1)
                {
                    g->hero2.bombs[j].isActive = 0;
                    g->hero2.bombs[j].local = 1;
                }
            }
        }
        g->hero2.put_bomb = 0;
    
}

int bomb_collision(Hero *h, Rectangle target)
{
    for (int i = 0; i < h->num_bombs; i++)
    {
        if(h->bombs[i].isActive){
            if (!(h->bombs[i].local) && CheckCollisionRecs(target, h->bombs[i].pos))
            {
                return 1;
            }
        }
    }
    return 0;
}

int hero_collision(Rectangle h1, Rectangle h2 ){
    if(CheckCollisionRecs(h1,h2)){
        return 1;
    }
    return 0;
}

int bombs_hipercollision(Hero *h,Rectangle *target){
    for(int i=0;i<h->num_bombs;i++){
        if(h->bombs[i].isActive && target!=&(h->bombs[i].pos) && (
            CheckCollisionRecs(*target,h->bombs[i].explosion_down) || CheckCollisionRecs(*target,h->bombs[i].explosion_up) 
        || CheckCollisionRecs(*target,h->bombs[i].explosion_left) || CheckCollisionRecs(*target,h->bombs[i].explosion_right))){
            return 1;
        }
    }
    return 0;
}

int barrier2_collision(Map *map, Rectangle target)
{
    for (int i = 0; i < map->num_barriers; i++)
    {
        if (CheckCollisionRecs(target, map->barriers[i]))
        {
            return 1;
        }
    }
    return 0;
}

int barrier_collision(Map *map, Rectangle target)
{
    for (int i = 0; i < map->num_barriers; i++)
    {
        if (CheckCollisionRecs(target, map->barriers[i]) || (!map->blocks[i].isBroken && CheckCollisionRecs(target,map->blocks[i].rect)))
        {
            return 1;
        }
    }
    return 0;
}

int CheckCollisionWithBlocks(Map *map, Hero *h) {
    for (int i = 0; i < 25; i++) {
        for(int j=0;j<5;j++){
            if(h->bombs[j].isActive){
                if (!map->blocks[i].isBroken && (CheckCollisionRecs(h->bombs[j].explosion_down, map->blocks[i].rect)|| 
                CheckCollisionRecs(h->bombs[j].explosion_up, map->blocks[i].rect)|| CheckCollisionRecs(h->bombs[j].explosion_right, map->blocks[i].rect)|| CheckCollisionRecs(h->bombs[j].explosion_left, map->blocks[i].rect))) {
                    return i;
                }
            }
        }
    }
    return -1;
}

int PegouEspecial1(Hero *h, Game *g){
    for(int i=0;i<3;i++){
        if(!g->maps[g->curr_map].esp1[i].foiUsado && CheckCollisionRecs(h->pos,g->maps[g->curr_map].esp1[i].spc)){
            return i;
        }
    }
    return -1;
}

int PegouEspecial2(Hero *h, Game *g){
    for(int i=0;i<3;i++){
        if(!g->maps[g->curr_map].esp2[i].foiUsado && CheckCollisionRecs(h->pos,g->maps[g->curr_map].esp2[i].spc)){
            return i;
        }
    }
    return -1;
}

int PegouEspecial3(Hero *h, Game *g){
    for(int i=0;i<3;i++){
        if(!g->maps[g->curr_map].esp3[i].foiUsado && CheckCollisionRecs(h->pos,g->maps[g->curr_map].esp3[i].spc)){
            return i;
        }
    }
    return -1;
}

void placar_final(Game *g){
    FILE *placar = fopen("arquivo/placar_final.txt","a");
    fprintf(placar,"Jogador: %s     Vitorias: %d\n",g->hero1.name,g->hero1.vitorias);
    fprintf(placar,"Jogador: %s     Derrotas: %d\n",g->hero1.name,g->hero1.derrotas);
    fprintf(placar,"Jogador: %s     Vitorias: %d\n",g->hero2.name,g->hero2.vitorias);
    fprintf(placar,"Jogador: %s     Derrotas: %d\n",g->hero2.name,g->hero2.derrotas);
    fclose(placar);
}

// Maps Setup
void map0_setup(Game *g)
{
 
    g->maps[0].num_barriers = 40;
    // Obstaculos
    for (int i = 0; i < 6; i++)
        {
            for (int j = 0; j < 6; j++)
            {
                g->maps[0].barriers[i * 6 + j] = (Rectangle){g->screenWidth * i / 7.058 + 85, g->screenHeight * j / 7.5 + 80, 50, 40};
            }
        }
    // Bordas
        g->maps[0].barriers[36] = (Rectangle){0, 0, SCREEN_BORDER, g->screenHeight};
        g->maps[0].barriers[37] = (Rectangle){0, 0, g->screenWidth, SCREEN_BORDER};
        g->maps[0].barriers[38] = (Rectangle){g->screenWidth - SCREEN_BORDER, 0, g->screenWidth, g->screenHeight};
        g->maps[0].barriers[39] = (Rectangle){0, g->screenHeight - SCREEN_BORDER, g->screenWidth, g->screenHeight};


        //Obstaculos Quebráveis
        g->maps[0].blocks[0].rect = (Rectangle) {135,80,64,40};
        g->maps[0].blocks[1].rect = (Rectangle) {362,80,64,40};
        g->maps[0].blocks[2].rect = (Rectangle) {588,80,64,40};
        g->maps[0].blocks[3].rect = (Rectangle) {248,160,64,40};
        g->maps[0].blocks[4].rect = (Rectangle) {475,160,64,40};
        g->maps[0].blocks[5].rect = (Rectangle) {135,240,64,40};
        g->maps[0].blocks[6].rect = (Rectangle) {362,240,64,40};
        g->maps[0].blocks[7].rect = (Rectangle) {588,240,64,40};
        g->maps[0].blocks[8].rect = (Rectangle) {248,320,64,40};
        g->maps[0].blocks[9].rect = (Rectangle) {475,320,64,40};
        g->maps[0].blocks[10].rect = (Rectangle) {135,400,64,40};
        g->maps[0].blocks[11].rect = (Rectangle) {362,400,64,40};
        g->maps[0].blocks[12].rect = (Rectangle) {588,400,64,40};
        g->maps[0].blocks[13].rect = (Rectangle) {248,480,64,40};
        g->maps[0].blocks[14].rect = (Rectangle) {475,480,64,40};
        for (int i = 0; i <= 14; i++)
        {
            g->maps[0].blocks[i].isBroken = 0;
        }
    //Especial
        for(int i=0;i<3;i++){
            //Aumenta Velocidade
                if(g->maps[0].esp1[i].foiUsado==0){
                g->maps[0].esp1[0].spc = (Rectangle) {155,90,15,15};
                g->maps[0].esp1[1].spc = (Rectangle) {495,490,15,15};
                g->maps[0].esp1[2].spc = (Rectangle) {155,250,15,15};
                g->maps[0].esp1[3].spc = (Rectangle) {382,330,15,15};
                }
                //Aumenta bombas
                if(g->maps[0].esp2[i].foiUsado==0){
                g->maps[0].esp2[0].spc = (Rectangle) {608,90,15,15};
                g->maps[0].esp2[1].spc = (Rectangle) {268,490,15,15};
                g->maps[0].esp2[2].spc = (Rectangle) {268,330,15,15};
                g->maps[0].esp2[3].spc = (Rectangle) {495,410,15,15};
                }
                //Aumenta raio das bombas
                if(g->maps[0].esp3[i].foiUsado==0){
               g->maps[0].esp3[0].spc = (Rectangle) {268,170,15,15};
                g->maps[0].esp3[1].spc = (Rectangle) {382,410,15,15};
                g->maps[0].esp3[2].spc = (Rectangle) {608,410,15,15};
                g->maps[0].esp3[3].spc = (Rectangle) {608,330,15,15};
                }
        }

        // Bombas Player1
        for (int i = 0; i < 10; i++)
        {
            g->hero1.bombs[i].isActive = 0;
            g->hero1.bombs[i].distance = 3;
        }
        // Bombas Player 2
        for (int j = 0; j < 10; j++)
        {
            g->hero2.bombs[j].isActive = 0;
            g->hero2.bombs[j].distance = 3;
        }
    
}

void map1_setup(Game *g)
{
 
        g->maps[1].num_barriers = 40;
        // Obstaculos
        for (int i = 0; i < 6; i++)
        {
            for (int j = 0; j < 6; j++)
            {
                g->maps[1].barriers[i * 6 + j] = (Rectangle){g->screenWidth * i / 7.058 + 85, g->screenHeight * j / 7.5 + 80, 50, 40};
            }
        }
        // Bordas
        g->maps[1].barriers[36] = (Rectangle){0, 0, SCREEN_BORDER, g->screenHeight};
        g->maps[1].barriers[37] = (Rectangle){0, 0, g->screenWidth, SCREEN_BORDER};
        g->maps[1].barriers[38] = (Rectangle){g->screenWidth - SCREEN_BORDER, 0, g->screenWidth, g->screenHeight};
        g->maps[1].barriers[39] = (Rectangle){0, g->screenHeight - SCREEN_BORDER, g->screenWidth, g->screenHeight};

        //Obstaculos Quebráveis
         g->maps[1].blocks[0].rect = (Rectangle) {135,80,64,40};
                    g->maps[1].blocks[1].rect = (Rectangle) {248,80,64,40};
                    g->maps[1].blocks[2].rect = (Rectangle) {475,80,64,40};
                    g->maps[1].blocks[3].rect = (Rectangle) {362,160,64,40};
                    g->maps[1].blocks[4].rect = (Rectangle) {588,160,64,40};
                    g->maps[1].blocks[5].rect = (Rectangle) {135,160,64,40};
                    g->maps[1].blocks[6].rect = (Rectangle) {248,240,64,40};
                    g->maps[1].blocks[7].rect = (Rectangle) {475,240,64,40};
                    g->maps[1].blocks[8].rect = (Rectangle) {248,320,64,40};
                    g->maps[1].blocks[9].rect = (Rectangle) {362,320,64,40};
                    g->maps[1].blocks[10].rect = (Rectangle) {588,320,64,40};
                    g->maps[1].blocks[11].rect = (Rectangle) {135,400,64,40};
                    g->maps[1].blocks[12].rect = (Rectangle) {362,400,64,40};
                    g->maps[1].blocks[13].rect = (Rectangle) {475,400,64,40};
                    g->maps[1].blocks[14].rect = (Rectangle) {588,400,64,40};
                    g->maps[1].blocks[15].rect = (Rectangle) {248,480,64,40};
                    g->maps[1].blocks[16].rect = (Rectangle) {475,480,64,40};
        for(int i=0;i<17;i++){
            g->maps[1].blocks[i].isBroken = 0;
        }

        //Especial
        for(int i=0;i<2;i++){
            //Aumenta Velocidade
            if(g->maps[1].esp1[i].foiUsado==0){
                g->maps[1].esp1[0].spc = (Rectangle) {495,90,15,15};
                g->maps[1].esp1[1].spc = (Rectangle) {495,490,15,15};
                g->maps[1].esp1[2].spc = (Rectangle) {268,330,15,15};
            }
                //Aumenta bombas
            if(g->maps[1].esp2[i].foiUsado==0){
                g->maps[1].esp2[0].spc = (Rectangle) {382,410,15,15};
                g->maps[1].esp2[1].spc = (Rectangle) {382,170,15,15};
                g->maps[1].esp2[2].spc = (Rectangle) {495,250,15,15};
            }
                //Aumenta raio das bombas
            if(g->maps[1].esp3[i].foiUsado==0){
                g->maps[1].esp3[0].spc = (Rectangle) {155,170,15,15};
                g->maps[1].esp3[1].spc = (Rectangle) {608,330,15,15};
                g->maps[1].esp3[2].spc = (Rectangle) {155,410,15,15};
            }
        }

        // Bombas Player1
        for (int i = 0; i < 10; i++)
        {
            g->hero1.bombs[i].isActive = 0;
            g->hero1.bombs[i].distance = 3;
        }
        // Bombas Player 2
        for (int j = 0; j < 10; j++)
        {
            g->hero2.bombs[j].isActive = 0;
            g->hero2.bombs[j].distance = 3;
        }
    
}
