# Pong
ping pong
import pygame, sys, time

# Runs imported module
pygame.init()
pygame.mixer.init()

# Constants
WIN_W = 920
WIN_H = 570
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
pygame.mixer.pre_init(44100, -16, 2, 2048)
pygame.display.set_mode()
intro_back = pygame.image.load("image/introBackground.jpg").convert()
intro_back = pygame.transform.scale(intro_back, (WIN_W, WIN_H))
game_back = pygame.image.load("image/background.jpg").convert()
game_back = pygame.transform.scale(game_back, (WIN_W, WIN_H))
outro_back = pygame.image.load("image/brick.jpg").convert()
outro_back = pygame.transform.scale(outro_back, (WIN_W, WIN_H))

sound = {}
sound["beep"] = pygame.mixer.Sound("Sounds/beep_converted.ogg")
sound["boom"] = pygame.mixer.Sound("Sounds/boom_converted.ogg")
sound["bop"] = pygame.mixer.Sound("Sounds/bop_converted.ogg")
sound["choose"] = pygame.mixer.Sound("Sounds/choose_converted.ogg")
sound["count"] = pygame.mixer.Sound("Sounds/count_converted.ogg")
sound["end"] = pygame.mixer.Sound("Sounds/end_converted.ogg")
sound["music"] = pygame.mixer.Sound("Sounds/music_converted.ogg")
sound["select"] = pygame.mixer.Sound("Sounds/select_converted.ogg")


class Paddle():
   def __init__(self, x, y):
       self.x = x
       self.y = y
       self.speed = 10
       self.paddle_width = 20
       self.paddle_height = 70
       self.image = pygame.image.load("image/paddle.png").convert()
       self.image = pygame.transform.scale(self.image, (self.paddle_width, self.paddle_height))
       self.score = 0
       self.rect = self.image.get_rect()
       self.rect = self.rect.move(x, y)

   def update(self, up, down):
       if up or down:
           if up:
               self.y -= self.speed
           if down:
               self.y += self.speed
       if self.y > WIN_H - self.paddle_height:
           self.y = WIN_H - self.paddle_height
       if self.y < 0:
           self.y = 0


class Ball():
   def __init__(self):
       self.speed = [5, 5]
       self.ball_width = 20
       self.ball_height = 20
       self.image = pygame.image.load("image/ball.png").convert()
       self.image = pygame.transform.scale(self.image, (self.ball_width, self.ball_height))
       self.Rect = pygame.Rect(WIN_W / 2, WIN_H / 2 - (self.ball_height / 2), self.ball_width, self.ball_height)

   def update(self, paddle_right, paddle_left):
       if self.Rect.top > WIN_H - self.Rect.height:
           self.speed[1] = -self.speed[1]
       if self.Rect.top < 0:
           self.speed[1] = -self.speed[1]
       '''if self.Rect.right > WIN_W:
           self.speed[0] = -self.speed[0]
       if self.Rect.left < 0:
           self.speed[0] = -self.speed[0]
       '''

       if self.Rect.top > paddle_right.y - 10 and self.Rect.bottom < paddle_right.y + paddle_right.paddle_height + 10:
           if self.Rect.x < paddle_right.x + 15 and self.Rect.x > paddle_right.x - 15:
               self.speed[0] = -self.speed[0]
       if self.Rect.top > paddle_left.y - 10 and self.Rect.bottom < paddle_left.y + paddle_left.paddle_height + 10:
           if self.Rect.x > paddle_left.x - 15 and self.Rect.x < paddle_left.x + 15:
               self.speed[0] = -self.speed[0]

       self.Rect = self.Rect.move(self.speed)


class Text():
   def __init__(self, size, text, x, y, color):
       self.font = pygame.font.Font(None, size)
       self.image = self.font.render(text, 1, color)
       self.rect = (x, y)


class Game():
   def __init__(self, paddle_right, paddle_left):
       self.intro = True
       self.play = False
       self.outro = False
       self.again = True
       self.moveUP_left = False
       self.moveDOWN_left = False
       self.moveUP_right = False
       self.moveDOWN_right = False
       self.fps = 60
       self.clock = pygame.time.Clock()
       self.beg_time = pygame.time.get_ticks()
       self.screen = pygame.display.set_mode((WIN_W, WIN_H), pygame.SRCALPHA)

       self.title = Text(100, "Pong", WIN_W / 2, 100, WHITE)
       self.click = Text(40, "-- Click Here --", WIN_W / 2, WIN_H / 2, WHITE)
       self.click_a = Text(40, "-- Click Here to Play Again --", WIN_W / 2, WIN_H - WIN_H / 10, WHITE)
       self.winner1 = Text(100, "Player 1 Wins", WIN_W / 2 - 120, WIN_H / 2, WHITE)
       self.winner2 = Text(100, "Player 2 Wins", WIN_W / 2 - 120, WIN_H / 2, WHITE)
       self.p1_score = Text(40, "Player 1 Score: " + str(paddle_left.score), 30, WIN_H / 15, WHITE)
       self.p2_score = Text(40, "Player 2 Score: " + str(paddle_right.score), WIN_W - 250, WIN_H / 15, WHITE)

   def restart(self, paddle_right, paddle_left, ball):
       self.screen.blit(game_back, (0, 0))

       time.sleep(1)
       ball.Rect.y = WIN_H / 2 - (ball.ball_height / 2)
       ball.Rect.x = WIN_W / 2

       paddle_left.y = (WIN_H / 2) - (paddle_left.paddle_height / 2)
       paddle_right.y = (WIN_H / 2) - (paddle_right.paddle_height / 2)

       self.screen.blit(self.p1_score.image, self.p1_score.rect)
       self.screen.blit(self.p2_score.image, self.p2_score.rect)
       self.screen.blit(paddle_right.image, (paddle_right.x, paddle_right.y))
       self.screen.blit(paddle_left.image, (paddle_left.x, paddle_left.y))
       self.screen.blit(ball.image, ball.Rect)

       pygame.display.flip()

       time.sleep(1)

   def update(self, paddle_right, paddle_left, ball):
       self.cur_time = pygame.time.get_ticks()
       self.blink = ((self.cur_time - self.beg_time) % 1000) < 500
       self.x = pygame.event.get()

       # Checks if window exit button pressed
       for event in self.x:
           if event.type == pygame.QUIT:
               sys.exit()

           # Keypresses
           if event.type == pygame.KEYDOWN:
               if event.key == pygame.K_UP:
                   self.moveUP_left = True
                   self.moveDOWN_left = False
               elif event.key == pygame.K_DOWN:
                   self.moveUP_left = False
                   self.moveDOWN_left = True

           if event.type == pygame.KEYUP:
               if event.key == pygame.K_UP:
                   self.moveUP_left = False
               elif event.key == pygame.K_DOWN:
                   self.moveDOWN_left = False

           if event.type == pygame.KEYDOWN:
               if event.key == pygame.K_w:
                   self.moveUP_right = True
                   self.moveDOWN_right = False
               elif event.key == pygame.K_s:
                   self.moveUP_right = False
                   self.moveDOWN_right = True

           if event.type == pygame.KEYUP:
               if event.key == pygame.K_w:
                   self.moveUP_right = False
               elif event.key == pygame.K_s:
                   self.moveDOWN_right = False

       # If ball moves off the screen
       if ball.Rect.right <= 0 - ball.Rect.width or ball.Rect.left >= WIN_W + ball.Rect.width:
           if ball.Rect.right <= 0:
               sound["count"].play()
               paddle_right.score += 1
           elif ball.Rect.left >= WIN_H + ball.Rect.width:
               sound["count"].play()
               paddle_left.score += 1
           self.p1_score = Text(40, "Player 1 Score: " + str(paddle_left.score), 30, WIN_H / 15, WHITE)
           self.p2_score = Text(40, "Player 2 Score: " + str(paddle_right.score), WIN_W - 250, WIN_H / 15, WHITE)
           self.restart(paddle_right, paddle_left, ball)

       if paddle_left.score == 3 or paddle_right.score == 3:
           self.play = False
           self.outro = True

       if self.play:
           self.screen.blit(self.p1_score.image, self.p1_score.rect)
           self.screen.blit(self.p2_score.image, self.p2_score.rect)


def main():
   pygame.display.set_caption('Pong')

   # Classes
   paddle_right = Paddle(30, WIN_H / 2 - 70 / 2)
   paddle_left = Paddle(WIN_W - 50, WIN_H / 2 - 70 / 2)
   ball = Ball()
   game = Game(paddle_right, paddle_left)

   while game.again:
       while game.intro:
           # Print background
           game.screen.blit(intro_back, (0, 0))

           game.screen.blit(game.title.image, game.title.rect)

           game.update(paddle_right, paddle_left, ball)

           # Blinking Text: Click here to start
           if game.blink:
               game.screen.blit(game.click.image, game.click.rect)

           for event in game.x:
               if event.type == pygame.QUIT:
                   sys.exit()
               elif event.type == pygame.MOUSEBUTTONDOWN or pygame.key.get_pressed()[pygame.K_RETURN] != 0:
                   sound["select"].play()
                   game.screen.blit(game.click.image, game.click.rect)
                   pygame.display.flip()
                   pygame.time.wait(1500)
                   game.intro = False
                   game.play = True

           game.clock.tick(game.fps)
           # Writes to main surface
           pygame.display.flip()

       # Gameplay
       while game.play:
           # printing to the main screen
           # paddle movement
           game.screen.blit(game_back, (0, 0))

           paddle_right.update(game.moveUP_right, game.moveDOWN_right)
           paddle_left.update(game.moveUP_left, game.moveDOWN_left)
           ball.update(paddle_right, paddle_left)
           game.update(paddle_right, paddle_left, ball)

           game.screen.blit(paddle_right.image, (paddle_right.x, paddle_right.y))
           game.screen.blit(paddle_left.image, (paddle_left.x, paddle_left.y))
           game.screen.blit(ball.image, ball.Rect)

           game.clock.tick(game.fps)
           # Writes to main surface
           pygame.display.flip()

       while game.outro:
           game.screen.blit(outro_back, (0, 0))

           if paddle_right.score == 3:
               game.screen.blit(game.winner2.image, game.winner2.rect)
           elif paddle_left.score == 3:
               game.screen.blit(game.winner1.image, game.winner1.rect)

           game.update(paddle_right, paddle_left, ball)

           if game.blink:
               game.screen.blit(game.click_a.image, game.click_a.rect)

           for event in game.x:
               if event.type == pygame.QUIT:
                   sys.exit()
               elif event.type == pygame.MOUSEBUTTONDOWN or pygame.key.get_pressed()[pygame.K_RETURN] != 0:
                   game.screen.blit(game.click_a.image, game.click_a.rect)
                   pygame.display.flip()
                   pygame.time.wait(1500)
                   game.play = True
                   game.outro = False
                   paddle_right.score = 0
                   paddle_left.score = 0

           game.clock.tick(game.fps)
           # Writes to main surface
           pygame.display.flip()

if __name__ == "__main__":
  main()

