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
        // --- ZMIENNE GLOBALNE GRY ---

        Timer t = new Timer();                  // zegar gry, dzięki niemu gra się odświeża co X ms
        List<Point> snake = new List<Point>();  // lista przechowująca wszystkie segmenty węża
        Point food;                             // pozycja jedzenia (x,y)
        int dirX = 1, dirY = 0;                 // kierunek ruchu (startowo w prawo)
        int size = 20;                          // rozmiar 1 kratki (w pikselach)
        Random r = new Random();                // generator losowych pozycji jedzenia

        // ✅👉 TODO 1: Tutaj dodaj zmienną przechowującą wynik (score)
        // (np. int score = 0;)

        public Form1()
        {
            InitializeComponent();

            this.DoubleBuffered = true;  // wygładza animację (brak migania ekranu)
            this.Width = 420;            // szerokość okna
            this.Height = 440;           // wysokość okna

            snake.Add(new Point(5, 5));  // startowa pozycja węża (jego "głowa")

            // losujemy pierwsze jedzenie
            food = new Point(r.Next(0, 20), r.Next(0, 20));

            // konfiguracja timera (pętli gry)
            t.Interval = 100;   // co 100ms gra wykonuje "krok"
            t.Tick += Game;     // przypisujemy funkcję Game jako akcję timera
            t.Start();          // startujemy grę

            this.KeyDown += KeyPush;  // nasłuch klawiatury (sterowanie strzałkami)
        }

        // --- GŁÓWNA PĘTLA GRY ---
        void Game(object sender, EventArgs e)
        {
            // nowa pozycja głowy = poprzednia pozycja + kierunek
            Point head = new Point(snake[0].X + dirX, snake[0].Y + dirY);

            // jeśli głowa wychodzi poza planszę lub dotknie siebie → koniec gry
            if (head.X < 0 || head.X > 19 || head.Y < 0 || head.Y > 19 || snake.Contains(head))
            {
                t.Stop();
                MessageBox.Show("Game Over");
                Close();
            }

            // dodajemy nową głowę na początek listy
            snake.Insert(0, head);

            // jeśli zjadł jedzenie → losujemy nowe
            if (head == food)
            {
                food = new Point(r.Next(0, 20), r.Next(0, 20));

                // ✅👉 TODO 2: Tutaj zwiększ wynik (score) o 1
                // (np. score++;)
            }
            else
                snake.RemoveAt(snake.Count - 1); // jeśli nie zjadł → usuwamy ogon (wąż się nie wydłuża)

            Invalidate(); // odśwież ekran → uruchomi OnPaint
        }

        // --- STEROWANIE STRZAŁKAMI ---
        void KeyPush(object sender, KeyEventArgs e)
        {
            if (e.KeyCode == Keys.Left)  { dirX = -1; dirY = 0; }
            if (e.KeyCode == Keys.Right) { dirX = 1;  dirY = 0; }
            if (e.KeyCode == Keys.Up)    { dirX = 0;  dirY = -1; }
            if (e.KeyCode == Keys.Down)  { dirX = 0;  dirY = 1; }
        }

        // --- RYSOWANIE NA EKRANIE ---
        protected override void OnPaint(PaintEventArgs e)
        {
            Graphics g = e.Graphics;

            // ✅👉 TODO 4: zmień kolor jabłka (np. Brushes.Pink, Brushes.Yellow, Brushes.Purple)
            g.FillRectangle(Brushes.Red, food.X * size, food.Y * size, size, size);

            // ✅👉 TODO 5: zmień kolor węża (np. Brushes.Blue, Brushes.Black, Brushes.Orange)
            foreach (var s in snake)
                g.FillRectangle(Brushes.Green, s.X * size, s.Y * size, size, size);

            // ✅👉 TODO 3: Tutaj wyświetl wynik na ekranie
            // (np. g.DrawString("Score: " + score, new Font("Arial", 12), Brushes.Black, 10, 10); )
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
