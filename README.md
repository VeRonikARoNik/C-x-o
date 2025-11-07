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

namespace SnakeWinForms
{
    enum Direction { Up, Down, Left, Right }

    public class SnakeForm : Form
    {
        // --- Gameplay config ---
        private int cell = 20;              // pixel size of a cell
        private int cols = 32;              // grid columns
        private int rows = 24;              // grid rows
        private int baseInterval = 120;     // ms
        private int minInterval = 55;       // ms
        private int speedupEvery = 50;      // points

        // --- State ---
        private readonly Timer timer = new Timer();
        private readonly Random rng = new Random();
        private List<Point> snake = new List<Point>();
        private Direction dir = Direction.Right;
        private bool grow = false;
        private bool paused = false;
        private bool gameOver = false;
        private Point food;
        private int score = 0;

        public SnakeForm()
        {
            Text = "Snake – WinForms";
            DoubleBuffered = true; // smooth drawing
            FormBorderStyle = FormBorderStyle.FixedSingle;
            MaximizeBox = false;
            BackColor = Color.FromArgb(24, 24, 24);
            ForeColor = Color.White;

            // Compute and set client size from grid
            ClientSize = new Size(cols * cell, rows * cell + 36); // + HUD strip

            // Timer
            timer.Interval = baseInterval;
            timer.Tick += (s, e) => GameTick();

            KeyPreview = true; // allow form to catch arrow keys
            KeyDown += OnKeyDown;

            ResetGame();
        }

        private void ResetGame()
        {
            score = 0;
            dir = Direction.Right;
            grow = false;
            paused = false;
            gameOver = false;
            timer.Interval = baseInterval;

            snake = new List<Point>();
            var start = new Point(cols / 2, rows / 2);
            snake.Add(start);
            snake.Add(new Point(start.X - 1, start.Y));
            snake.Add(new Point(start.X - 2, start.Y));

            SpawnFood();
            timer.Start();
            Invalidate();
        }

        private void SpawnFood()
        {
            while (true)
            {
                var p = new Point(rng.Next(1, cols - 1), rng.Next(2, rows - 1)); // avoid HUD row
                if (!snake.Contains(p)) { food = p; return; }
            }
        }

        private void GameTick()
        {
            if (paused || gameOver) return;

            var head = snake[0];
            var next = dir switch
            {
                Direction.Left  => new Point(head.X - 1, head.Y),
                Direction.Right => new Point(head.X + 1, head.Y),
                Direction.Up    => new Point(head.X, head.Y - 1),
                Direction.Down  => new Point(head.X, head.Y + 1),
                _ => head
            };

            // Wall collision (keep 0..cols-1, 1..rows-1 to leave row 0 as HUD)
            if (next.X <= 0 || next.Y <= 1 || next.X >= cols - 1 || next.Y >= rows - 1)
            {
                EndGame();
                return;
            }

            // Self collision (allow tail move unless growing)
            for (int i = 0; i < snake.Count - (grow ? 0 : 1); i++)
                if (snake[i] == next) { EndGame(); return; }

            // Move
            snake.Insert(0, next);
            if (next == food)
            {
                score += 10;
                grow = true;
                SpawnFood();
                // Speed up every N points
                if (score % speedupEvery == 0)
                    timer.Interval = Math.Max(minInterval, timer.Interval - 5);
            }

            if (!grow)
                snake.RemoveAt(snake.Count - 1);
            else
                grow = false;

            Invalidate();
        }

        private void EndGame()
        {
            gameOver = true;
            timer.Stop();
            Invalidate();
        }

        private void OnKeyDown(object? sender, KeyEventArgs e)
        {
            if (gameOver)
            {
                if (e.KeyCode == Keys.R) ResetGame();
                if (e.KeyCode == Keys.Escape) Close();
                return;
            }

            switch (e.KeyCode)
            {
                case Keys.Up:    if (dir != Direction.Down) dir = Direction.Up; break;
                case Keys.Down:  if (dir != Direction.Up) dir = Direction.Down; break;
                case Keys.Left:  if (dir != Direction.Right) dir = Direction.Left; break;
                case Keys.Right: if (dir != Direction.Left) dir = Direction.Right; break;
                case Keys.Space:
                case Keys.P:
                    paused = !paused; if (!paused) Focus(); break;
                case Keys.R:
                    ResetGame(); break;
                case Keys.Escape:
                    Close(); break;
            }
        }

        protected override void OnPaint(PaintEventArgs e)
        {
            base.OnPaint(e);
            var g = e.Graphics;
            g.SmoothingMode = System.Drawing.Drawing2D.SmoothingMode.None;

            // HUD (top strip at grid row 0)
            var hudRect = new Rectangle(0, 0, ClientSize.Width, cell + 4);
            using (var hudBg = new SolidBrush(Color.FromArgb(32, 32, 32))) g.FillRectangle(hudBg, hudRect);
            using (var pen = new Pen(Color.FromArgb(64, 64, 64))) g.DrawLine(pen, 0, cell + 4, ClientSize.Width, cell + 4);
            using (var f = new Font("Segoe UI", 10, FontStyle.Bold))
            using (var hudBrush = new SolidBrush(Color.White))
            {
                g.DrawString($"Score: {score}   Speed: {1000 / timer.Interval:0} cells/s   [Arrows] Move  [Space/P] Pause  [R] Restart  [Esc] Quit", f, hudBrush, 8, 6);
            }

            // Playfield background
            var fieldRect = new Rectangle(0, cell + 5, cols * cell, (rows - 1) * cell);
            using (var bg = new SolidBrush(Color.FromArgb(18, 18, 18))) g.FillRectangle(bg, fieldRect);

            // Walls
            using (var wall = new SolidBrush(Color.FromArgb(60, 60, 60)))
            {
                // top and bottom
                g.FillRectangle(wall, 0, cell + 5, cols * cell, cell); // top wall line
                g.FillRectangle(wall, 0, (rows - 1) * cell + 5, cols * cell, cell); // bottom wall line
                // left and right
                g.FillRectangle(wall, 0, cell + 5, cell, (rows - 1) * cell);
                g.FillRectangle(wall, (cols - 1) * cell, cell + 5, cell, (rows - 1) * cell);
            }

            // Food
            DrawCell(g, food, Color.Gold);

            // Snake
            for (int i = 0; i < snake.Count; i++)
            {
                var color = (i == 0) ? Color.Lime : Color.FromArgb(200, 230, 230, 230);
                DrawCell(g, snake[i], color);
            }

            if (paused)
                DrawCenterBanner(g, "PAUSED\nPress Space to resume");

            if (gameOver)
                DrawCenterBanner(g, $"GAME OVER\nScore: {score}\nPress R to restart");
        }

        private void DrawCell(Graphics g, Point p, Color color)
        {
            int x = p.X * cell;
            int y = p.Y * cell + 5; // + HUD offset
            var rect = new Rectangle(x + 1, y + 1, cell - 2, cell - 2);
            using (var b = new SolidBrush(color)) g.FillRectangle(b, rect);
        }

        private void DrawCenterBanner(Graphics g, string text)
        {
            using var fmt = new StringFormat { Alignment = StringAlignment.Center, LineAlignment = StringAlignment.Center };
            using var font = new Font("Segoe UI", 20, FontStyle.Bold);
            using var small = new Font("Segoe UI", 10, FontStyle.Regular);
            var lines = text.Split('\n');
            var center = new Rectangle(0, cell + 5, cols * cell, (rows - 1) * cell);

            // Semi-transparent overlay
            using (var overlay = new SolidBrush(Color.FromArgb(120, 0, 0, 0))) g.FillRectangle(overlay, center);

            // Draw lines stacked
            float y = center.Top + center.Height / 2f - (lines.Length - 1) * 18;
            for (int i = 0; i < lines.Length; i++)
            {
                using var brush = new SolidBrush(Color.White);
                g.DrawString(lines[i], i == 0 ? font : small, brush, new RectangleF(center.Left, y, center.Width, 40), fmt);
                y += (i == 0 ? 44 : 22);
            }
        }

        [STAThread]
        public static void Main()
        {
            ApplicationConfiguration.Initialize();
            Application.Run(new SnakeForm());
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
