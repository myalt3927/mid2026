```C
#include "raylib.h"
#include <math.h>

static Rectangle paddle = {390, 600, 120, 14};
static Vector2 ball, speed;
static bool broken[70], launched;
static int lives, left;
static const Color colors[] = {RED, ORANGE, YELLOW, GREEN, SKYBLUE, BLUE, PURPLE};

static float Clamp(float x, float min, float max) { return fmaxf(min, fminf(x, max)); }
static Rectangle Brick(int i) { return (Rectangle){38 + i % 10 * 83, 126 + i / 10 * 32, 77, 24}; }
static void Label(const char *s, int y, int size) { DrawText(s, (900 - MeasureText(s, size)) / 2, y, size, WHITE); }

// 전체 재시작 또는 공만 패들 위로 되돌리기
static void Reset(bool all)
{
    if (all) {
        for (int i = 0; i < 70; i++) broken[i] = false;
        lives = 3; left = 70; paddle.x = 390;
    }
    ball = (Vector2){paddle.x + 60, 591};
    speed = (Vector2){180, -360};
    launched = false;
}

// 공 이동과 벽·패들·블록 충돌
static void MoveBall(float dt)
{
    Vector2 prev = ball;
    ball.x += speed.x * dt; ball.y += speed.y * dt;
    if (ball.x < 32) { ball.x = 32; speed.x = fabsf(speed.x); }
    if (ball.x > 868) { ball.x = 868; speed.x = -fabsf(speed.x); }
    if (ball.y < 98) { ball.y = 98; speed.y = fabsf(speed.y); }
    if (speed.y > 0 && CheckCollisionCircleRec(ball, 8, paddle)) {
        float angle = Clamp((ball.x - paddle.x - 60) / 60, -1, 1);
        ball.y = 592;
        speed = (Vector2){420 * sinf(angle), -420 * cosf(angle)};
    }
    for (int i = 0; i < 70; i++) {
        Rectangle b = Brick(i);
        if (broken[i] || !CheckCollisionCircleRec(ball, 8, b)) continue;
        broken[i] = true; left--;
        if (prev.x + 8 <= b.x || prev.x - 8 >= b.x + b.width) speed.x *= -1;
        else speed.y *= -1;
        ball = prev;
        return;
    }
    if (ball.y > 648) { lives--; Reset(false); }
}

int main(void)
{
    InitWindow(900, 680, "Rainbow Breaker");
    SetTargetFPS(120);
    Reset(true);
    while (!WindowShouldClose()) {
        float dt = fminf(GetFrameTime(), 1.0f / 30);
        if (IsKeyPressed(KEY_R)) Reset(true);
        if (lives && left) {
            int dir = (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D))
                    - (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A));
            if (dir) paddle.x += dir * 600 * dt;
            else if (GetMouseDelta().x != 0) paddle.x = GetMouseX() - 60;
            paddle.x = Clamp(paddle.x, 24, 756);
            if (!launched) {
                Reset(false);
                launched = IsKeyPressed(KEY_SPACE) || IsMouseButtonPressed(MOUSE_BUTTON_LEFT);
            }
            for (int i = 0; i < 4 && launched && left; i++) MoveBall(dt / 4);
        }
        BeginDrawing();
        ClearBackground((Color){14, 18, 32, 255});
        Label(TextFormat("RAINBOW BREAKER    SCORE %i    LIVES %i", (70 - left) * 10, lives), 30, 24);
        DrawRectangleLines(24, 90, 852, 550, DARKGRAY);
        for (int i = 0; i < 70; i++)
            if (!broken[i]) DrawRectangleRounded(Brick(i), 0.25f, 4, colors[i / 10]);
        DrawRectangleRounded(paddle, 1, 8, SKYBLUE);
        DrawCircleV(ball, 8, WHITE);
        Label("MOVE: Mouse / A D / Arrows    LAUNCH: Space / Click    R: Restart", 653, 16);
        if (!lives || !left) {
            DrawRectangle(25, 91, 850, 548, Fade(BLACK, 0.75f));
            Label(left ? "GAME OVER" : "YOU WIN!", 315, 48);
            Label("Press R to play again", 390, 24);
        } else if (!launched) Label("SPACE / CLICK TO LAUNCH", 435, 24);
        EndDrawing();
    }
    CloseWindow();
    return 0;
}

```
