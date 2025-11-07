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
using System.Windows.Forms;

namespace SnakeSimple
{
    public partial class Form1 : Form
    {
        Timer t = new Timer();
        List<Point> snake = new List<Point>();
        Point food;
        int dirX = 1, dirY = 0;
        int size = 20;
        Random r = new Random();

        // ✅ TODO 1: Dodaj tutaj zmienną przechowującą wynik (score)
        // ODPOWIEDŹ:
        // int score = 0;

        public Form1()
        {
            InitializeComponent();

            this.DoubleBuffered = true;
            this.ClientSize = new Size(20 * size, 20 * size);

            snake.Add(new Point(5, 5));
            food = NewFoodPosition();

            t.Interval = 100;
            t.Tick += Game;
            t.Start();

            this.KeyDown += KeyPush;
        }

        Point NewFoodPosition()
        {
            Point p;
            do
            {
                p = new Point(r.Next(0, 20), r.Next(0, 20));
            } while (snake.Contains(p));
            return p;
        }

        void Game(object sender, EventArgs e)
        {
            Point head = new Point(snake[0].X + dirX, snake[0].Y + dirY);

            if (head.X < 0 || head.X > 19 || head.Y < 0 || head.Y > 19 || snake.Contains(head))
            {
                t.Stop();

                // ✅ TODO 2: W MessageBox wypisz wynik gracza
                // ODPOWIEDŹ:
                // MessageBox.Show("Game Over\nScore: " + score);

                MessageBox.Show("Game Over");

                Close();
                return;
            }

            snake.Insert(0, head);

            if (head == food)
            {
                // ✅ TODO 3: Zwiększ wynik o 1
                // ODPOWIEDŹ:
                // score++;

                food = NewFoodPosition();
            }
            else
            {
                snake.RemoveAt(snake.Count - 1);
            }

            Invalidate();
        }

        void KeyPush(object sender, KeyEventArgs e)
        {
            if (e.KeyCode == Keys.Left && dirX != 1) { dirX = -1; dirY = 0; }
            if (e.KeyCode == Keys.Right && dirX != -1) { dirX = 1; dirY = 0; }
            if (e.KeyCode == Keys.Up && dirY != 1) { dirX = 0; dirY = -1; }
            if (e.KeyCode == Keys.Down && dirY != -1) { dirX = 0; dirY = 1; }
        }

        protected override void OnPaint(PaintEventArgs e)
        {
            base.OnPaint(e);
            Graphics g = e.Graphics;

            // ✅ TODO 4: Zmień kolor jabłka
            // ODPOWIEDŹ:
            // g.FillRectangle(Brushes.Yellow, food.X * size, food.Y * size, size, size);

            g.FillRectangle(Brushes.Red, food.X * size, food.Y * size, size, size);

            // ✅ TODO 5: Zmień kolor węża
            // ODPOWIEDŹ:
            // g.FillRectangle(Brushes.Black, s.X * size, s.Y * size, size, size);

            foreach (var s in snake)
                g.FillRectangle(Brushes.Green, s.X * size, s.Y * size, size, size);

            // ✅ TODO 6: Wyświetl wynik na ekranie
            // ODPOWIEDŹ:
            // g.DrawString("Score: " + score, new Font("Arial", 12, FontStyle.Bold), Brushes.Black, 5, 5);
        }
    }
}

```
Quiz game

label1, label2,label3,  radioButton1, radioButton2, radioButton3, radioButton4, button1, button2

```
using System;
using System.Windows.Forms;

namespace QuizApp
{
    public partial class Form1 : Form
    {
        int questionIndex = 0;
        int score = 0;

        string[] questions = {
            "1. Ile to 2 + 2?",
            "2. Które zwierzę jest największe?",
            "3. W którym roku był pierwszy lot w kosmos?"
        };

        string[,] answers = {
            { "3", "4", "5", "6" },
            { "Słoń", "Rekin", "Wieloryb", "Lew" },
            { "1957", "1961", "1969", "1975" }
        };

        int[] correctAnswers = { 1, 2, 1 }; // indeksy poprawnych odpowiedzi (0-based)

        public Form1()
        {
            InitializeComponent();
            LoadQuestion();
        }

        private void LoadQuestion()
        {
            // Ustaw pytanie
            label1.Text = questions[questionIndex];
            label2.Text = "";
            label3.Text = $"Wynik: {score}";

            // Ustaw odpowiedzi
            radioButton1.Text = answers[questionIndex, 0];
            radioButton2.Text = answers[questionIndex, 1];
            radioButton3.Text = answers[questionIndex, 2];
            radioButton4.Text = answers[questionIndex, 3];

            // Odznacz radio buttony
            radioButton1.Checked = radioButton2.Checked =
                radioButton3.Checked = radioButton4.Checked = false;
        }

        private void button1_Click(object sender, EventArgs e)
        {
            int selected = -1;
            if (radioButton1.Checked) selected = 0;
            else if (radioButton2.Checked) selected = 1;
            else if (radioButton3.Checked) selected = 2;
            else if (radioButton4.Checked) selected = 3;

            if (selected == -1)
            {
                MessageBox.Show("Wybierz odpowiedź!");
                return;
            }

            if (selected == correctAnswers[questionIndex])
            {
                label2.Text = "✅ Poprawna odpowiedź!";
                score++;
            }
            else
            {
                label2.Text = "❌ Zła odpowiedź!";
            }

            label3.Text = $"Wynik: {score}";
        }

        private void button2_Click(object sender, EventArgs e)
        {
            questionIndex++;
            if (questionIndex < questions.Length)
                LoadQuestion();
            else
                MessageBox.Show($"Koniec quizu! Twój wynik: {score}/{questions.Length}");
        }
    }
}

```

⭐ zmienić teksty na własne
⭐ dodać więcej niż 1 pytanie
⭐ zmienić temat quizu (np. Minecraft, sport, psy itd.)
