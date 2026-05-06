import pygame
import random
import sys

# Константы игры
SCREEN_WIDTH = 640
SCREEN_HEIGHT = 480
GRID_SIZE = 20
GRID_WIDTH = SCREEN_WIDTH // GRID_SIZE
GRID_HEIGHT = SCREEN_HEIGHT // GRID_SIZE

# Цвета
BACKGROUND_COLOR = (0, 0, 0)
SNAKE_COLOR = (0, 255, 0)
APPLE_COLOR = (255, 0, 0)

# Направления движения
UP = (0, -1)
DOWN = (0, 1)
LEFT = (-1, 0)
RIGHT = (1, 0)


class GameObject:
    """Базовый класс для игровых объектов."""

    def __init__(self, position, body_color):
        """
        Инициализирует игровой объект.

        Args:
            position (tuple): Координаты объекта (x, y).
            body_color (tuple): Цвет объекта в формате RGB.
        """
        self.position = position
        self.body_color = body_color

    def draw(self, surface):
        """
        Отрисовывает объект на поверхности.
        Должен быть переопределен в дочерних классах.
        
        Args:
            surface: Поверхность Pygame для отрисовки.
        """
        pass


class Apple(GameObject):
    """Класс, описывающий яблоко."""

    def __init__(self):
        """Инициализирует яблоко со случайной позицией и красным цветом."""
        super().__init__((0, 0), APPLE_COLOR)
        self.randomize_position()

    def randomize_position(self):
        """Устанавливает случайную позицию яблока в пределах игрового поля."""
        self.position = (
            random.randint(0, GRID_WIDTH - 1) * GRID_SIZE,
            random.randint(0, GRID_HEIGHT - 1) * GRID_SIZE
        )

    def draw(self, surface):
        """
        Отрисовывает яблоко на заданной поверхности.

        Args:
            surface: Поверхность Pygame для отрисовки.
        """
        rect = pygame.Rect(
            (self.position[0], self.position[1]),
            (GRID_SIZE, GRID_SIZE)
        )
        pygame.draw.rect(surface, self.body_color, rect)


class Snake(GameObject):
    """Класс, описывающий змейку."""

    def __init__(self):
        """Инициализирует змейку в центре экрана."""
        # Начальная позиция - центр экрана
        start_x = (GRID_WIDTH // 2) * GRID_SIZE
        start_y = (GRID_HEIGHT // 2) * GRID_SIZE
        
        super().__init__((start_x, start_y), SNAKE_COLOR)
        
        self.length = 1
        self.positions = [self.position]
        self.direction = RIGHT
        self.next_direction = None

    def get_head_position(self):
        """
        Возвращает позицию головы змейки.

        Returns:
            tuple: Координаты головы змейки.
        """
        return self.positions[0]

    def update_direction(self):
        """Обновляет текущее направление движения на следующее, если оно задано."""
        if self.next_direction:
            # Проверка, чтобы змейка не могла развернуться на 180 градусов
            if (self.next_direction[0] * -1, self.next_direction[1] * -1) != self.direction:
                self.direction = self.next_direction
            self.next_direction = None

    def move(self):
        """
        Обновляет позицию змейки.
        Добавляет новую голову и удаляет хвост, если длина не изменилась.
        Реализует проход сквозь стены.
        """
        cur = self.get_head_position()
        x, y = self.direction
        
        # Вычисляем новые координаты головы
        new_x = (cur[0] + (x * GRID_SIZE)) % SCREEN_WIDTH
        new_y = (cur[1] + (y * GRID_SIZE)) % SCREEN_HEIGHT
        
        new_position = (new_x, new_y)

        # Проверка на столкновение с самим собой
        if len(self.positions) > 1 and new_position in self.positions:
            self.reset()
        else:
            # Вставляем новую голову в начало списка
            self.positions.insert(0, new_position)
            
            # Если длина списка больше реальной длины змейки, удаляем хвост
            if len(self.positions) > self.length:
                self.positions.pop()

    def reset(self):
        """Сбрасывает змейку в начальное состояние."""
        self.length = 1
        start_x = (GRID_WIDTH // 2) * GRID_SIZE
        start_y = (GRID_HEIGHT // 2) * GRID_SIZE
        self.positions = [(start_x, start_y)]
        self.direction = RIGHT
        self.next_direction = None

    def draw(self, surface):
        """
        Отрисовывает змейку на заданной поверхности.

        Args:
            surface: Поверхность Pygame для отрисовки.
        """
        for p in self.positions:
            rect = pygame.Rect(
                (p[0], p[1]),
                (GRID_SIZE, GRID_SIZE)
            )
            pygame.draw.rect(surface, self.body_color, rect)
            # Опционально: обводка сегментов для лучшей видимости
            pygame.draw.rect(surface, BACKGROUND_COLOR, rect, 1)


def handle_keys(snake):
    """
    Обрабатывает нажатия клавиш для управления змейкой.

    Args:
        snake: Экземпляр класса Snake.
    """
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_UP:
                snake.next_direction = UP
            elif event.key == pygame.K_DOWN:
                snake.next_direction = DOWN
            elif event.key == pygame.K_LEFT:
                snake.next_direction = LEFT
            elif event.key == pygame.K_RIGHT:
                snake.next_direction = RIGHT


def main():
    """Основная функция игры."""
    pygame.init()
    
    # Создание окна
    screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
    pygame.display.set_caption("Змейка")
    
    # Часы для контроля FPS
    clock = pygame.time.Clock()
    
    # Создание объектов
    snake = Snake()
    apple = Apple()
    
    # Основной игровой цикл
    while True:
        # Обработка событий
        handle_keys(snake)
        
        # Обновление логики
        snake.update_direction()
        snake.move()
        
        # Проверка: съела ли змейка яблоко
        if snake.get_head_position() == apple.position:
            snake.length += 1
            apple.randomize_position()
            # Убедимся, что яблоко не появилось внутри змейки
            while apple.position in snake.positions:
                apple.randomize_position()
        
        # Отрисовка
        screen.fill(BACKGROUND_COLOR)
        snake.draw(screen)
        apple.draw(screen)
        
        # Обновление экрана
        pygame.display.update()
        
        # Ограничение скорости игры (20 кадров в секунду)
        clock.tick(20)


if __name__ == '__main__':
    main()
