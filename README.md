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





Snake


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
            this.KeyDown += GameForm_KeyDown;

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
                case Dir.Up:    next = new Point(head.X, head.Y - 1); break;
                case Dir.Down:  next = new Point(head.X, head.Y + 1); break;
                case Dir.Left:  next = new Point(head.X - 1, head.Y); break;
                case Dir.Right: next = new Point(head.X + 1, head.Y); break;
            }

            if (next.X < 0 || next.X >= COLS || next.Y < 0 || next.Y >= ROWS || snake.Contains(next))
            {
                alive = false;
                Invalidate();
                var res = MessageBox.Show($"Koniec gry! Wynik: {score}\nZagrać ponownie?", "Żmijka",
                    MessageBoxButtons.YesNo, MessageBoxIcon.Information);
                if (res == DialogResult.Yes) ResetGame();
                return;
            }

            snake.Insert(0, next);

            if (next == food)
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

            if (e.KeyCode == Keys.Up    && dir != Dir.Down)  dir = Dir.Up;
            if (e.KeyCode == Keys.Down  && dir != Dir.Up)    dir = Dir.Down;
            if (e.KeyCode == Keys.Left  && dir != Dir.Right) dir = Dir.Left;
            if (e.KeyCode == Keys.Right && dir != Dir.Left)  dir = Dir.Right;
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

            string info = alive ? $"Wynik: {score}  |  Sterowanie: strzałki, P=pauza"
                                : $"Koniec gry! Wynik: {score}  |  Spacja = nowa gra";
            using var font = new Font("Segoe UI",  nine: false);
            using var brush2 = new SolidBrush(Color.Black);
            g.DrawString(info, font, brush2, 4, ROWS * CELL + 6);
        }
    }
}

```


```
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Linq;
using System.Windows.Forms;

namespace QuizABC
{
    public class QuizForm : Form
    {
        private class Question
        {
            public string Text { get; set; }
            public string[] Answers { get; set; } = new string[4];
            public int CorrectIndex { get; set; }
        }

        // UI
        private Label lblTitle;
        private Label lblQuestion;
        private RadioButton[] options = new RadioButton[4];
        private Button btnCheck;
        private Button btnNext;
        private Label lblFeedback;
        private Label lblScore;

        // State
        private List<Question> quiz;
        private int idx = 0;
        private int score = 0;
        private bool checkedThis = false;

        public QuizForm()
        {
            Text = "Quiz wiedzy (ABC)";
            StartPosition = FormStartPosition.CenterScreen;
            MinimumSize = new Size(640, 420);
            Font = new Font("Segoe UI", 10);
            KeyPreview = true;

            BuildUi();
            BuildQuiz();      // tu definiujesz pytania
            LoadQuestion(0);  // start
        }

        private void BuildUi()
        {
            var root = new TableLayoutPanel
            {
                Dock = DockStyle.Fill,
                ColumnCount = 1,
                RowCount = 5,
                Padding = new Padding(16),
            };
            root.RowStyles.Add(new RowStyle(SizeType.AutoSize));        // tytuł
            root.RowStyles.Add(new RowStyle(SizeType.AutoSize));        // pytanie
            root.RowStyles.Add(new RowStyle(SizeType.Percent, 100));    // odpowiedzi
            root.RowStyles.Add(new RowStyle(SizeType.AutoSize));        // feedback
            root.RowStyles.Add(new RowStyle(SizeType.AutoSize));        // przyciski / wynik
            Controls.Add(root);

            lblTitle = new Label
            {
                Text = "QUIZ — wybierz jedną odpowiedź",
                Font = new Font(Font, FontStyle.Bold),
                AutoSize = true,
                Margin = new Padding(0, 0, 0, 6)
            };
            root.Controls.Add(lblTitle, 0, 0);

            lblQuestion = new Label
            {
                Text = "Treść pytania...",
                Font = new Font(Font.FontFamily, 12, FontStyle.Regular),
                AutoSize = true,
                MaximumSize = new Size(1000, 0), // zawijanie
                Margin = new Padding(0, 0, 0, 10)
            };
            root.Controls.Add(lblQuestion, 0, 1);

            var answersPanel = new TableLayoutPanel
            {
                Dock = DockStyle.Fill,
                ColumnCount = 1,
                RowCount = 4,
            };
            for (int i = 0; i < 4; i++)
            {
                answersPanel.RowStyles.Add(new RowStyle(SizeType.Percent, 25));
                var rb = new RadioButton
                {
                    Text = $"Opcja {i + 1}",
                    AutoSize = true,
                    Margin = new Padding(0, 6, 0, 6)
                };
                options[i] = rb;
                answersPanel.Controls.Add(rb, 0, i);
            }
            root.Controls.Add(answersPanel, 0, 2);

            lblFeedback = new Label
            {
                Text = " ",
                AutoSize = true,
                ForeColor = Color.DimGray,
                Margin = new Padding(0, 6, 0, 6)
            };
            root.Controls.Add(lblFeedback, 0, 3);

            var bottom = new FlowLayoutPanel
            {
                FlowDirection = FlowDirection.LeftToRight,
                Dock = DockStyle.Fill,
                AutoSize = true
            };

            btnCheck = new Button { Text = "Sprawdź", AutoSize = true };
            btnCheck.Click += (s, e) => CheckAnswer();

            btnNext = new Button { Text = "Następne pytanie", AutoSize = true, Enabled = false };
            btnNext.Click += (s, e) => NextQuestion();

            lblScore = new Label { Text = "Wynik: 0 pkt", AutoSize = true, Margin = new Padding(16, 6, 0, 0) };

            bottom.Controls.Add(btnCheck);
            bottom.Controls.Add(btnNext);
            bottom.Controls.Add(lblScore);

            root.Controls.Add(bottom, 0, 4);

            // Enter = Sprawdź, Ctrl+Enter = Następne
            this.KeyDown += (s, e) =>
            {
                if (e.KeyCode == Keys.Enter && !e.Control)
                {
                    e.SuppressKeyPress = true;
                    if (btnCheck.Enabled) CheckAnswer();
                }
                else if (e.KeyCode == Keys.Enter && e.Control)
                {
                    e.SuppressKeyPress = true;
                    if (btnNext.Enabled) NextQuestion();
                }
            };
        }

        private void BuildQuiz()
        {
            // PRZYKŁADOWE PYTANIA – podmień na swoje
            quiz = new List<Question>
            {
                new Question {
                    Text = "Który typ w C# przechowuje liczby całkowite?",
                    Answers = new[] { "string", "int", "double", "bool" },
                    CorrectIndex = 1
                },
                new Question {
                    Text = "Co zwraca operator '==' w C#?",
                    Answers = new[] { "Liczbę całkowitą", "Znak", "Wartość bool (true/false)", "Nic" },
                    CorrectIndex = 2
                },
                new Question {
                    Text = "Która pętla wykona się co najmniej raz?",
                    Answers = new[] { "for", "while", "do-while", "foreach" },
                    CorrectIndex = 2
                },
                new Question {
                    Text = "Jaki jest domyślny modyfikator dostępu dla pól klasy w C#?",
                    Answers = new[] { "public", "private", "protected", "internal" },
                    CorrectIndex = 1
                },
            };

            // (opcjonalnie) losowa kolejność
            // var rnd = new Random();
            // quiz = quiz.OrderBy(_ => rnd.Next()).ToList();
        }

        private void LoadQuestion(int i)
        {
            if (i >= quiz.Count)
            {
                EndQuiz();
                return;
            }

            var q = quiz[i];
            lblQuestion.Text = $"Pytanie {i + 1}/{quiz.Count}: {q.Text}";
            for (int k = 0; k < 4; k++)
            {
                options[k].Text = q.Answers[k];
                options[k].Checked = false;
                options[k].Enabled = true;
            }
            lblFeedback.Text = " ";
            lblFeedback.ForeColor = Color.DimGray;

            btnCheck.Enabled = true;
            btnNext.Enabled = false;
            checkedThis = false;
        }

        private void CheckAnswer()
        {
            if (checkedThis) return;

            int selected = Array.FindIndex(options, rb => rb.Checked);
            if (selected == -1)
            {
                MessageBox.Show("Wybierz odpowiedź.", "Uwaga", MessageBoxButtons.OK, MessageBoxIcon.Information);
                return;
            }

            var q = quiz[idx];
            bool ok = selected == q.CorrectIndex;

            if (ok)
            {
                score += 1;
                lblFeedback.Text = "✅ Dobrze!";
                lblFeedback.ForeColor = Color.SeaGreen;
            }
            else
            {
                lblFeedback.Text = $"❌ Źle. Poprawna odpowiedź: {q.Answers[q.CorrectIndex]}";
                lblFeedback.ForeColor = Color.IndianRed;
            }

            lblScore.Text = $"Wynik: {score} pkt";

            // zablokuj zmiany odpowiedzi
            foreach (var rb in options) rb.Enabled = false;

            btnCheck.Enabled = false;
            btnNext.Enabled = true;
            checkedThis = true;
        }

        private void NextQuestion()
        {
            if (!checkedThis)
            {
                MessageBox.Show("Najpierw kliknij 'Sprawdź'.", "Uwaga", MessageBoxButtons.OK, MessageBoxIcon.Information);
                return;
            }
            idx++;
            LoadQuestion(idx);
        }

        private void EndQuiz()
        {
            foreach (var rb in options) rb.Enabled = false;
            btnCheck.Enabled = false;
            btnNext.Enabled = false;

            lblQuestion.Text = "Koniec quizu!";
            lblFeedback.Text = $"Twój wynik: {score} / {quiz.Count}";
            lblFeedback.ForeColor = Color.DodgerBlue;

            var res = MessageBox.Show($"Koniec quizu!\nWynik: {score}/{quiz.Count}\n\nCzy chcesz zagrać ponownie?",
                "Quiz", MessageBoxButtons.YesNo, MessageBoxIcon.Information);
            if (res == DialogResult.Yes)
            {
                idx = 0; score = 0;
                lblScore.Text = "Wynik: 0 pkt";
                LoadQuestion(0);
            }
        }
    }
}


```
