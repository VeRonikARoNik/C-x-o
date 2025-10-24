# C-x-o
O&amp;X


```
using System;
using System.Linq;
using System.Windows.Forms;

namespace KolkoKrzyzykTextboxy
{
    public partial class Form1 : Form
    {
        char current = 'X';
        bool active = true;

        public Form1()
        {
            InitializeComponent();
            this.Text = "Kółko i krzyżyk";
        }

        private void button1_Click(object sender, EventArgs e)
        {
            if (!active) return;

            // Wszystkie pola
            TextBox[] t = { textBox1, textBox2, textBox3,
                            textBox4, textBox5, textBox6,
                            textBox7, textBox8, textBox9 };

            // Znajdź pole z wpisanym X lub O, które nie jest zablokowane
            var chosen = t.FirstOrDefault(tb => !tb.ReadOnly && !string.IsNullOrWhiteSpace(tb.Text));
            if (chosen == null)
            {
                MessageBox.Show("Wpisz X lub O w wolne pole i kliknij 'Zatwierdź ruch'.");
                return;
            }

            char ch = char.ToUpper(chosen.Text[0]);
            if (ch != 'X' && ch != 'O')
            {
                MessageBox.Show("Dozwolone są tylko X lub O.");
                chosen.Clear();
                return;
            }

            if (ch != current)
            {
                MessageBox.Show($"Teraz ruch gracza: {current}");
                chosen.Clear();
                return;
            }

            chosen.Text = ch.ToString();
            chosen.ReadOnly = true;

            // Sprawdź wygraną
            if (Winner(t))
            {
                active = false;
                MessageBox.Show($"Wygrał {current}!");
                return;
            }

            // Sprawdź remis
            if (t.All(tb => tb.ReadOnly))
            {
                active = false;
                MessageBox.Show("Remis!");
                return;
            }

            // Zmiana gracza
            current = (current == 'X') ? 'O' : 'X';
        }

        private void button2_Click(object sender, EventArgs e)
        {
            // Nowa gra
            TextBox[] t = { textBox1, textBox2, textBox3,
                            textBox4, textBox5, textBox6,
                            textBox7, textBox8, textBox9 };

            foreach (var tb in t)
            {
                tb.Clear();
                tb.ReadOnly = false;
                tb.BackColor = System.Drawing.SystemColors.Window;
            }

            current = 'X';
            active = true;
        }

        bool Winner(TextBox[] t)
        {
            int[,] lines =
            {
                {0,1,2}, {3,4,5}, {6,7,8},
                {0,3,6}, {1,4,7}, {2,5,8},
                {0,4,8}, {2,4,6}
            };

            for (int i = 0; i < lines.GetLength(0); i++)
            {
                int a = lines[i,0], b = lines[i,1], c = lines[i,2];
                if (!string.IsNullOrEmpty(t[a].Text) &&
                    t[a].Text == t[b].Text && t[b].Text == t[c].Text)
                {
                    // Podświetlenie zwycięskiej linii
                    t[a].BackColor = t[b].BackColor = t[c].BackColor = System.Drawing.Color.LightGreen;
                    return true;
                }
            }
            return false;
        }
    }
}


```





Snake game


```

using System;
using System.Collections.Generic;
using System.Drawing;
using System.Linq;
using System.Windows.Forms;

namespace SnakeMinimal
{
    public class GameForm : Form
    {
        const int CELL = 20;
        const int COLS = 20;
        const int ROWS = 20;

        enum Dir { Up, Down, Left, Right }
        List<Point> snake = new List<Point>();
        Dir dir = Dir.Right;
        Point food;
        Timer timer = new Timer();
        Random rng = new Random();
        bool alive = true;
        int score = 0;

        public GameForm()
        {
            Text = "Żmijka — najprostsza wersja";
            ClientSize = new Size(COLS * CELL, ROWS * CELL + 30);
            BackColor = Color.White;
            DoubleBuffered = true;
            KeyPreview = true;

            timer.Interval = 120;
            timer.Tick += (s, e) => TickGame();
            KeyDown += GameForm_KeyDown;

            ResetGame();
        }

        void ResetGame()
        {
            alive = true;
            score = 0;
            dir = Dir.Right;
            snake.Clear();
            var start = new Point(COLS / 2, ROWS / 2);
            snake.Add(start);
            snake.Add(new Point(start.X - 1, start.Y));
            snake.Add(new Point(start.X - 2, start.Y));
            SpawnFood();
            timer.Start();
            Invalidate();
        }

        void SpawnFood()
        {
            while (true)
            {
                var p = new Point(rng.Next(0, COLS), rng.Next(0, ROWS));
                if (!snake.Contains(p)) { food = p; return; }
            }
        }

        void TickGame()
        {
            if (!alive) { timer.Stop(); return; }

            var head = snake[0];
            var next = head;
            switch (dir)
            {
                case Dir.Up: next = new Point(head.X, head.Y - 1); break;
                case Dir.Down: next = new Point(head.X, head.Y + 1); break;
                case Dir.Left: next = new Point(head.X - 1, head.Y); break;
                case Dir.Right: next = new Point(head.X + 1, head.Y); break;
            }

            bool outOfBounds = next.X < 0 || next.X >= COLS || next.Y < 0 || next.Y >= ROWS;
            bool willGrow = (next == food);

            // Jeśli nie rośniemy, ogon się zaraz usunie — więc go pomijamy przy sprawdzaniu kolizji.
            bool hitsSelf = willGrow
                ? snake.Contains(next)
                : snake.Take(snake.Count - 1).Contains(next);

            if (outOfBounds || hitsSelf)
            {
                alive = false;
                Invalidate();
                var res = MessageBox.Show($"Koniec gry! Wynik: {score}\nZagrać ponownie?", "Żmijka",
                    MessageBoxButtons.YesNo, MessageBoxIcon.Information);
                if (res == DialogResult.Yes) ResetGame();
                return;
            }

            // Ruch
            snake.Insert(0, next);

            if (willGrow)
            {
                score += 10;
                SpawnFood();
            }
            else
            {
                snake.RemoveAt(snake.Count - 1);
            }

            Invalidate();
        }

        void GameForm_KeyDown(object sender, KeyEventArgs e)
        {
            if (!alive && e.KeyCode == Keys.Space)
            {
                ResetGame();
                return;
            }

            if (e.KeyCode == Keys.Up && dir != Dir.Down) dir = Dir.Up;
            if (e.KeyCode == Keys.Down && dir != Dir.Up) dir = Dir.Down;
            if (e.KeyCode == Keys.Left && dir != Dir.Right) dir = Dir.Left;
            if (e.KeyCode == Keys.Right && dir != Dir.Left) dir = Dir.Right;

            if (e.KeyCode == Keys.P)
            {
                if (timer.Enabled) timer.Stop(); else timer.Start();
            }
        }

        protected override void OnPaint(PaintEventArgs e)
        {
            base.OnPaint(e);
            var g = e.Graphics;

            using var foodBrush = new SolidBrush(Color.IndianRed);
            g.FillRectangle(foodBrush, food.X * CELL, food.Y * CELL, CELL, CELL);

            for (int i = 0; i < snake.Count; i++)
            {
                var p = snake[i];
                Rectangle rect = new Rectangle(p.X * CELL, p.Y * CELL, CELL, CELL);
                using var brush = new SolidBrush(i == 0 ? Color.SeaGreen : Color.MediumSeaGreen);
                g.FillRectangle(brush, rect);
            }

            string info = alive
                ? $"Wynik: {score}  |  Sterowanie: strzałki, P=pauza"
                : $"Koniec gry! Wynik: {score}  |  Spacja = nowa gra";

            using var font = new Font("Segoe UI", 9f, FontStyle.Regular, GraphicsUnit.Point);
            using var brush2 = new SolidBrush(Color.Black);
            g.DrawString(info, font, brush2, 4, ROWS * CELL + 6);
        }
    }
}


```
Quiz game

| Typ         | Nazwa (Name)    | Tekst (Text)     | Funkcja                           |
| ----------- | --------------- | ---------------- | --------------------------------- |
| Label       | `labelQuestion` | Treść pytania    | wyświetla aktualne pytanie        |
| RadioButton | `radioA`        | Odpowiedź A      | pierwsza odpowiedź                |
| RadioButton | `radioB`        | Odpowiedź B      | druga odpowiedź                   |
| RadioButton | `radioC`        | Odpowiedź C      | trzecia odpowiedź                 |
| RadioButton | `radioD`        | Odpowiedź D      | czwarta odpowiedź                 |
| Label       | `labelResult`   | *(puste)*        | wyświetla wynik odpowiedzi        |
| Label       | `labelScore`    | Wynik: 0         | pokazuje liczbę zdobytych punktów |
| Button      | `buttonCheck`   | Sprawdź          | sprawdza odpowiedź                |
| Button      | `buttonNext`    | Następne pytanie | przechodzi do kolejnego pytania   |


```
using System;
using System.Collections.Generic;
using System.Windows.Forms;

namespace QuizABC
{
    public partial class Form1 : Form
    {
        // struktura pytania
        class Question
        {
            public string Text;
            public string[] Answers;
            public int Correct;
        }

        List<Question> quiz = new List<Question>();
        int index = 0;
        int score = 0;

        public Form1()
        {
            InitializeComponent();
            BuildQuiz();
            LoadQuestion(0);
        }

        void BuildQuiz()
        {
            quiz.Add(new Question
            {
                Text = "Który typ danych w C# przechowuje liczby całkowite?",
                Answers = new[] { "string", "int", "double", "bool" },
                Correct = 1
            });

            quiz.Add(new Question
            {
                Text = "Który operator służy do porównywania wartości?",
                Answers = new[] { "=", "==", "equals", "!=" },
                Correct = 1
            });

            quiz.Add(new Question
            {
                Text = "Która pętla wykona się co najmniej raz?",
                Answers = new[] { "for", "while", "do-while", "foreach" },
                Correct = 2
            });

            quiz.Add(new Question
            {
                Text = "Jaką wartość logiczną ma warunek: 5 > 10?",
                Answers = new[] { "true", "false", "error", "null" },
                Correct = 1
            });
        }

        void LoadQuestion(int i)
        {
            if (i >= quiz.Count)
            {
                MessageBox.Show($"Koniec quizu!\nTwój wynik: {score}/{quiz.Count}", "Quiz");
                ResetQuiz();
                return;
            }

            var q = quiz[i];
            labelQuestion.Text = q.Text;
            radioA.Text = q.Answers[0];
            radioB.Text = q.Answers[1];
            radioC.Text = q.Answers[2];
            radioD.Text = q.Answers[3];

            radioA.Checked = radioB.Checked = radioC.Checked = radioD.Checked = false;
            labelResult.Text = "";
        }

        private void buttonCheck_Click(object sender, EventArgs e)
        {
            if (index >= quiz.Count) return;

            var q = quiz[index];
            int selected = -1;

            if (radioA.Checked) selected = 0;
            else if (radioB.Checked) selected = 1;
            else if (radioC.Checked) selected = 2;
            else if (radioD.Checked) selected = 3;

            if (selected == -1)
            {
                MessageBox.Show("Zaznacz odpowiedź!", "Uwaga");
                return;
            }

            if (selected == q.Correct)
            {
                score++;
                labelResult.Text = "✅ Dobrze!";
                labelResult.ForeColor = System.Drawing.Color.Green;
            }
            else
            {
                labelResult.Text = $"❌ Źle! Poprawna: {q.Answers[q.Correct]}";
                labelResult.ForeColor = System.Drawing.Color.Red;
            }

            labelScore.Text = $"Wynik: {score}";
            buttonCheck.Enabled = false;
        }

        private void buttonNext_Click(object sender, EventArgs e)
        {
            index++;
            buttonCheck.Enabled = true;
            LoadQuestion(index);
        }

        void ResetQuiz()
        {
            index = 0;
            score = 0;
            labelScore.Text = "Wynik: 0";
            buttonCheck.Enabled = true;
            LoadQuestion(0);
        }
    }
}

```
