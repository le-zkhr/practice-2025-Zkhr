# **Исследование предметной области**
Для создания видеоигры тетрис мне понадобились:
- знания языка C++
- графической библиотеки SFML. Она проста в использовании и быстро интегрируется в проект, поэтому я выбрал именно её.
- знания правил игры классического Тетриса

# **Этапы создания технологии**
1. Сначала я определил размеры игрового окна, а также элементов игры в файле Global.hpp:
```C++
#pragma once

constexpr unsigned char CELL_SIZE = 8;
constexpr unsigned char CLEAR_EFFECT_DURATION = 8;
constexpr unsigned char COLUMNS = 10;
constexpr unsigned char LINES_TO_INCREASE_SPEED = 2;
constexpr unsigned char MOVE_SPEED = 4;
constexpr unsigned char ROWS = 20;
constexpr unsigned char SCREEN_RESIZE = 4;
constexpr unsigned char SOFT_DROP_SPEED = 4;
constexpr unsigned char START_FALL_SPEED = 32;
constexpr unsigned short FRAME_DURATION = 16667;

struct Position
{
	char x;
	char y;
};
```

![Так выглядит игровое поле](/Users/zkhr/Documents/Education/Практика/Tetris_Game/Images/Empty.png)

2. Затем я создал матрицу игрового поля в Main.cpp:
```C++
	std::vector<std::vector<unsigned char>> matrix(COLUMNS, std::vector<unsigned char>(ROWS));
```

![Так выглядит игровая матрица](/Users/zkhr/Documents/Education/Практика/Tetris_Game/Images/Matrix.png)

3. После я создал класс Tetromino, где создал метод, для его движения вниз и метод возобновления его падения:
```C++
bool Tetromino::move_down(const std::vector<std::vector<unsigned char>>& i_matrix)
{
	for (Position& mino : minos)
	{
		Will we go outside the matrix if we move down?
		if (ROWS == 1 + mino.y)
		{
			return 0;
		}

		Will we hit another tetromino if we move down?
		if (0 < i_matrix[mino.x][1 + mino.y])
		{
			return 0;
		}
	}

	Move the tetromino down
	for (Position& mino : minos)
	{
		mino.y++;
	}

	Return that everything is okay
	return 1;
}

bool Tetromino::reset(unsigned char i_shape, const std::vector<std::vector<unsigned char>>& i_matrix)
{
	Reset the variables
	rotation = 0;
	shape = i_shape;

	minos = get_tetromino(shape, COLUMNS / 2, 1);

	for (Position& mino : minos)
	{
		if (0 < i_matrix[mino.x][mino.y])
		{
			Return that we can't reset because there's a tetromino at the spawn location
			return 0;
		}
	}

	Return that everything is fine
	return 1;
}
```

А также метод обновления матрицы:
```C++
void Tetromino::update_matrix(std::vector<std::vector<unsigned char>>& i_matrix)
{
	Putting the tetromino to the matrix
	for (Position& mino : minos)
	{
		if (0 > mino.y)
		{
			continue;
		}

		i_matrix[mino.x][mino.y] = 1 + shape;
	}
}
```

![Так выглядят первые тетрамино на поле](/Users/zkhr/Documents/Education/Практика/Tetris_Game/Images/Reds.png)

Благодаря этим методам тетрамино (квадрат пока что) двигаюстя вниз и появляются наверху для повторного падения.

После этого я добавил методы ***move_left, move_right, hard_drop, move_down, Tetromino::rotate*** - все они отвечают за движения тетрамино, за возможность быстро их ставить и вращать, основываясь на точке-пивоте.

А благодаря методу get_shape и остальной логики в этом же классе мы получили разнообразные тетрамины для спавна их сверху.

## **Таким образом, вся логика тетрамино (вращение, движение, генерация, форма, цвета и тд.) находятся в классе Tetramino.**

4. Вспомогательный класс GetTetromino служит для генерации тетрамино и их корректировки при спавне. А чтобы тетрамино корректно двигались, если вращаются вплотную к стенкам, я создал класс GetWallKickData, коротый обрабатывает данный случай.

![Так выглядят раскрашенные тетрамино](/Users/zkhr/Documents/Education/Практика/Tetris_Game/Images/Coloring.png)

5. После я добавил экран сбоку, показывающий следующие тетрамино в main.cpp, а затем выровнял их по центру доп. окна:
```C++
if (FRAME_DURATION > lag)
			{
				unsigned char clear_cell_size = static_cast<unsigned char>(2 * round(0.5f * CELL_SIZE * (clear_effect_timer / static_cast<float>(CLEAR_EFFECT_DURATION))));

				sf::RectangleShape cell(sf::Vector2f(CELL_SIZE - 1, CELL_SIZE - 1));
                
				sf::RectangleShape preview_border(sf::Vector2f(5 * CELL_SIZE, 5 * CELL_SIZE));
				preview_border.setFillColor(sf::Color(0, 0, 0));
				preview_border.setOutlineThickness(-1);
				preview_border.setPosition(CELL_SIZE * (1.5f * COLUMNS - 2.5f), CELL_SIZE * (0.25f * ROWS - 2.5f));

				window.clear();

				for (unsigned char a = 0; a < COLUMNS; a++)
				{
					for (unsigned char b = 0; b < ROWS; b++)
					{
						if (0 == clear_lines[b])
						{
							cell.setPosition(static_cast<float>(CELL_SIZE * a), static_cast<float>(CELL_SIZE * b));

							if (1 == game_over && 0 < matrix[a][b])
							{
								cell.setFillColor(cell_colors[8]);
							}
							else
							{
								cell.setFillColor(cell_colors[matrix[a][b]]);
							}

							window.draw(cell);
						}
					}
				}

				cell.setFillColor(cell_colors[8]);

				if (0 == game_over)
				{
					for (Position& mino : tetromino.get_ghost_minos(matrix))
					{
						cell.setPosition(static_cast<float>(CELL_SIZE * mino.x), static_cast<float>(CELL_SIZE * mino.y));

						window.draw(cell);
					}

					cell.setFillColor(cell_colors[1 + tetromino.get_shape()]);
				}

				for (Position& mino : tetromino.get_minos())
				{
					cell.setPosition(static_cast<float>(CELL_SIZE * mino.x), static_cast<float>(CELL_SIZE * mino.y));
					
					window.draw(cell);
				}

				for (unsigned char a = 0; a < COLUMNS; a++)
				{
					for (unsigned char b = 0; b < ROWS; b++)
					{
						if (1 == clear_lines[b])
						{
							cell.setFillColor(cell_colors[0]);
							cell.setPosition(static_cast<float>(CELL_SIZE * a), static_cast<float>(CELL_SIZE * b));
							cell.setSize(sf::Vector2f(CELL_SIZE - 1, CELL_SIZE - 1));

							window.draw(cell);

							cell.setFillColor(sf::Color(255, 255, 255));
							cell.setPosition(floor(CELL_SIZE * (0.5f + a) - 0.5f * clear_cell_size), floor(CELL_SIZE * (0.5f + b) - 0.5f * clear_cell_size));
							cell.setSize(sf::Vector2f(clear_cell_size, clear_cell_size));

							window.draw(cell);
						}
					}
				}

				cell.setFillColor(cell_colors[1 + next_shape]);
				cell.setSize(sf::Vector2f(CELL_SIZE - 1, CELL_SIZE - 1));

				window.draw(preview_border);

				for (Position& mino : get_tetromino(next_shape, static_cast<unsigned char>(1.5f * COLUMNS), static_cast<unsigned char>(0.25f * ROWS)))
				{
					Shifting the tetromino to the center of the preview border
					unsigned short next_tetromino_x = CELL_SIZE * mino.x;
					unsigned short next_tetromino_y = CELL_SIZE * mino.y;

					if (0 == next_shape)
					{
						next_tetromino_y += static_cast<unsigned char>(round(0.5f * CELL_SIZE));
					}
					else if (3 != next_shape)
					{
						next_tetromino_x -= static_cast<unsigned char>(round(0.5f * CELL_SIZE));
					}

					cell.setPosition(next_tetromino_x, next_tetromino_y);

					window.draw(cell);
				}

				draw_text(static_cast<unsigned short>(CELL_SIZE * (0.5f + COLUMNS)), static_cast<unsigned short>(0.5f * CELL_SIZE * ROWS), "Lines:" + std::to_string(lines_cleared) + "\nSpeed:" + std::to_string(START_FALL_SPEED / current_fall_speed) + 'x', window);
				
				window.display();
			}
```

![Так выглядит боковое окно](/Users/zkhr/Documents/Education/Практика/Tetris_Game/Images/PredictionWindow.png)

6. Я также добавил анимацию исчезновения строки, если каждая ее клетка занята частью тетрамино:
```C++
if (0 == clear_effect_timer)
			{
				if (0 == game_over)
				{
					if (0 == rotate_pressed)
					{
						If the C is pressed
						if (1 == sf::Keyboard::isKeyPressed(sf::Keyboard::C))
						{
							Rotation key is pressed!
							rotate_pressed = 1;

							Do a barrel roll
							tetromino.rotate(1, matrix);
						} Else, if the Z is pressed
						else if (1 == sf::Keyboard::isKeyPressed(sf::Keyboard::Z))
						{
							Rotation key is pressed!
							rotate_pressed = 1;

							Do a barrel roll but to the other side!
							tetromino.rotate(0, matrix);
						}
					}

					if (0 == move_timer)
					{
						If the Left is pressed
						if (1 == sf::Keyboard::isKeyPressed(sf::Keyboard::Left))
						{
							Reset the move timer
							move_timer = 1;

							Move the tetromino to the left
							tetromino.move_left(matrix);
						}
						else if (1 == sf::Keyboard::isKeyPressed(sf::Keyboard::Right))
						{
							move_timer = 1;
							tetromino.move_right(matrix);
						}
					}
					else
					{
						move_timer = (1 + move_timer) % MOVE_SPEED;
					}

					if (0 == hard_drop_pressed)
					{
						if (1 == sf::Keyboard::isKeyPressed(sf::Keyboard::Space))
						{
							hard_drop_pressed = 1;
							fall_timer = current_fall_speed;

							tetromino.hard_drop(matrix);
						}
					}

					if (0 == soft_drop_timer)
					{
						if (1 == sf::Keyboard::isKeyPressed(sf::Keyboard::Down))
						{
							if (1 == tetromino.move_down(matrix))
							{
								fall_timer = 0;
								soft_drop_timer = 1;
							}
						}
					}
					else
					{
						soft_drop_timer = (1 + soft_drop_timer) % SOFT_DROP_SPEED;
					}

					if (current_fall_speed == fall_timer)
					{
						if (0 == tetromino.move_down(matrix))
						{
							tetromino.update_matrix(matrix);
							for (unsigned char a = 0; a < ROWS; a++)
							{
								bool clear_line = 1;

								for (unsigned char b = 0; b < COLUMNS; b++)
								{
									if (0 == matrix[b][a])
									{
										clear_line = 0;

										break;
									}
								}

								if (1 == clear_line)
								{
									lines_cleared++;

									clear_effect_timer = CLEAR_EFFECT_DURATION;

									clear_lines[a] = 1;

									if (0 == lines_cleared % LINES_TO_INCREASE_SPEED)
									{
										current_fall_speed = std::max<unsigned char>(SOFT_DROP_SPEED, current_fall_speed - 1);
									}
								}
							}

							If the effect timer is over
							if (0 == clear_effect_timer)
							{
								game_over = 0 == tetromino.reset(next_shape, matrix);

								next_shape = static_cast<unsigned char>(shape_distribution(random_engine));
							}
						}

						fall_timer = 0;
					}
					else
					{
						fall_timer++;
					}
				}
				else if (1 == sf::Keyboard::isKeyPressed(sf::Keyboard::Enter))
				{
					We set everything to 0
					game_over = 0;
					hard_drop_pressed = 0;
					rotate_pressed = 0;

					lines_cleared = 0;

					current_fall_speed = START_FALL_SPEED;
					fall_timer = 0;
					move_timer = 0;
					soft_drop_timer = 0;

					for (std::vector<unsigned char>& a : matrix)
					{
						std::fill(a.begin(), a.end(), 0);
					}
				}
			}
			else
			{
				clear_effect_timer--;

				If the effect timer is between 1 and -1
				if (0 == clear_effect_timer)
				{
					for (unsigned char a = 0; a < ROWS; a++)
					{
						if (1 == clear_lines[a])
						{
							for (unsigned char b = 0; b < COLUMNS; b++)
							{
								Set the cell to 0 (empty) (the absence of existence)
								matrix[b][a] = 0;

								for (unsigned char c = a; 0 < c; c--)
								{
									matrix[b][c] = matrix[b][c - 1];
									matrix[b][c - 1] = 0;
								}
							}
						}
					}

					game_over = 0 == tetromino.reset(next_shape, matrix);

					next_shape = static_cast<unsigned char>(shape_distribution(random_engine));

					Clear the clear lines array
					std::fill(clear_lines.begin(), clear_lines.end(), 0);
				}
			}

			if (FRAME_DURATION > lag)
			{
				unsigned char clear_cell_size = static_cast<unsigned char>(2 * round(0.5f * CELL_SIZE * (clear_effect_timer / static_cast<float>(CLEAR_EFFECT_DURATION))));

				sf::RectangleShape cell(sf::Vector2f(CELL_SIZE - 1, CELL_SIZE - 1));
                
				sf::RectangleShape preview_border(sf::Vector2f(5 * CELL_SIZE, 5 * CELL_SIZE));
				preview_border.setFillColor(sf::Color(0, 0, 0));
				preview_border.setOutlineThickness(-1);
				preview_border.setPosition(CELL_SIZE * (1.5f * COLUMNS - 2.5f), CELL_SIZE * (0.25f * ROWS - 2.5f));

				window.clear();

				for (unsigned char a = 0; a < COLUMNS; a++)
				{
					for (unsigned char b = 0; b < ROWS; b++)
					{
						if (0 == clear_lines[b])
						{
							cell.setPosition(static_cast<float>(CELL_SIZE * a), static_cast<float>(CELL_SIZE * b));

							if (1 == game_over && 0 < matrix[a][b])
							{
								cell.setFillColor(cell_colors[8]);
							}
							else
							{
								cell.setFillColor(cell_colors[matrix[a][b]]);
							}

							window.draw(cell);
						}
					}
				}

				cell.setFillColor(cell_colors[8]);

				if (0 == game_over)
				{
					for (Position& mino : tetromino.get_ghost_minos(matrix))
					{
						cell.setPosition(static_cast<float>(CELL_SIZE * mino.x), static_cast<float>(CELL_SIZE * mino.y));

						window.draw(cell);
					}

					cell.setFillColor(cell_colors[1 + tetromino.get_shape()]);
				}

				for (Position& mino : tetromino.get_minos())
				{
					cell.setPosition(static_cast<float>(CELL_SIZE * mino.x), static_cast<float>(CELL_SIZE * mino.y));
					
					window.draw(cell);
				}

				for (unsigned char a = 0; a < COLUMNS; a++)
				{
					for (unsigned char b = 0; b < ROWS; b++)
					{
						if (1 == clear_lines[b])
						{
							cell.setFillColor(cell_colors[0]);
							cell.setPosition(static_cast<float>(CELL_SIZE * a), static_cast<float>(CELL_SIZE * b));
							cell.setSize(sf::Vector2f(CELL_SIZE - 1, CELL_SIZE - 1));

							window.draw(cell);

							cell.setFillColor(sf::Color(255, 255, 255));
							cell.setPosition(floor(CELL_SIZE * (0.5f + a) - 0.5f * clear_cell_size), floor(CELL_SIZE * (0.5f + b) - 0.5f * clear_cell_size));
							cell.setSize(sf::Vector2f(clear_cell_size, clear_cell_size));

							window.draw(cell);
						}
					}
				}

				cell.setFillColor(cell_colors[1 + next_shape]);
				cell.setSize(sf::Vector2f(CELL_SIZE - 1, CELL_SIZE - 1));

				window.draw(preview_border);

				for (Position& mino : get_tetromino(next_shape, static_cast<unsigned char>(1.5f * COLUMNS), static_cast<unsigned char>(0.25f * ROWS)))
				{
					Shifting the tetromino to the center of the preview border
					unsigned short next_tetromino_x = CELL_SIZE * mino.x;
					unsigned short next_tetromino_y = CELL_SIZE * mino.y;

					if (0 == next_shape)
					{
						next_tetromino_y += static_cast<unsigned char>(round(0.5f * CELL_SIZE));
					}
					else if (3 != next_shape)
					{
						next_tetromino_x -= static_cast<unsigned char>(round(0.5f * CELL_SIZE));
					}

					cell.setPosition(next_tetromino_x, next_tetromino_y);

					window.draw(cell);
				}

				draw_text(static_cast<unsigned short>(CELL_SIZE * (0.5f + COLUMNS)), static_cast<unsigned short>(0.5f * CELL_SIZE * ROWS), "Lines:" + std::to_string(lines_cleared) + "\nSpeed:" + std::to_string(START_FALL_SPEED / current_fall_speed) + 'x', window);
				
				window.display();
			}
```

Также я добавил функцию, которая окрашивает все тетрамино в серый при проигрыше.

7. Затем я добавил тетрамино-призрак, который будет находиться внизу и показывать, куда упадет текущий тетрамино:
```C++
void Tetromino::hard_drop(const std::vector<std::vector<unsigned char>>& i_matrix)
{
	minos = get_ghost_minos(i_matrix);
}

std::vector<Position> Tetromino::get_ghost_minos(const std::vector<std::vector<unsigned char>>& i_matrix)
{
	bool keep_falling = 1;

	unsigned char total_movement = 0;

	std::vector<Position> ghost_minos = minos;

	while (1 == keep_falling)
	{
		total_movement++;

		for (Position& mino : minos)
		{
			if (ROWS == total_movement + mino.y)
			{
				keep_falling = 0;

				break;
			}

			if (0 > total_movement + mino.y)
			{
				continue;
			}
			else if (0 < i_matrix[mino.x][total_movement + mino.y])
			{
				keep_falling = 0;

				break;
			}
		}
	}

	for (Position& mino : ghost_minos)
	{
		mino.y += total_movement - 1;
	}

	return ghost_minos;
}
```

![Так выглядит анимация исчезновения линии и тетрамино-призрак](/Users/zkhr/Documents/Education/Практика/Tetris_Game/Images/Ghost.png)

8. После я добавил изменения скорости падения тетрамино со временем:
```C++
if (0 == move_timer)
					{
						If the Left is pressed
						if (1 == sf::Keyboard::isKeyPressed(sf::Keyboard::Left))
						{
							Reset the move timer
							move_timer = 1;

							Move the tetromino to the left
							tetromino.move_left(matrix);
						}
						else if (1 == sf::Keyboard::isKeyPressed(sf::Keyboard::Right))
						{
							move_timer = 1;
							tetromino.move_right(matrix);
						}
					}
					else
					{
						move_timer = (1 + move_timer) % MOVE_SPEED;
					}

					if (0 == hard_drop_pressed)
					{
						if (1 == sf::Keyboard::isKeyPressed(sf::Keyboard::Space))
						{
							hard_drop_pressed = 1;
							fall_timer = current_fall_speed;

							tetromino.hard_drop(matrix);
						}
					}

					if (0 == soft_drop_timer)
					{
						if (1 == sf::Keyboard::isKeyPressed(sf::Keyboard::Down))
						{
							if (1 == tetromino.move_down(matrix))
							{
								fall_timer = 0;
								soft_drop_timer = 1;
							}
						}
					}
					else
					{
						soft_drop_timer = (1 + soft_drop_timer) % SOFT_DROP_SPEED;
					}
```

![Так выглядит анимация исчезновения линии и тетрамино-призрак](/Users/zkhr/Documents/Education/Практика/Tetris_Game/Images/SpeedUp.png)

9. Для добавления очков мне потребовалось добавить шрифт текста, а также внести изменения в код:
```C++
	draw_text(static_cast<unsigned short>(CELL_SIZE * (0.5f + COLUMNS)), static_cast<unsigned short>(0.5f * CELL_SIZE * ROWS), "Lines:" + std::to_string(lines_cleared) + "\nSpeed:" + std::to_string(START_FALL_SPEED / current_fall_speed) + 'x', window);
```

Сам же текст находится в ***/Resources/Images/Font.png***

10. Рефакторинг и очистка кода заняла значительную часть времени, но за счет нее удалось уменьшить количество кода и сделать его более оптимизированным в некоторых местах.