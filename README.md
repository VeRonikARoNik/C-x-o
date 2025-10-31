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
// Jeśli wolisz krócej, odkomentuj alias i używaj "Timer":
// using Timer = System.Windows.Forms.Timer;

namespace WinFormsApp3
{
    public partial class Form1 : Form
    {
        enum Dir { Right, Down, Left, Up }

        private readonly Panel canvas = new Panel();
        private readonly Label hud = new Label();
        private readonly System.Windows.Forms.Timer gameTimer = new System.Windows.Forms.Timer();
        // Jeśli używasz aliasu powyżej, możesz napisać: private readonly Timer gameTimer = new Timer();

        private readonly List<Point> snake = new List<Point>();
        private Point food;
        private Dir dir = Dir.Right;
        private int cell = 20;
        private int score = 0;
        private bool isGameOver = false;
        private readonly Random rng = new Random();

        public Form1()
        {
            InitializeComponent();

            // Okno
            Text = "Snake — C# WinForms";
            ClientSize = new Size(640, 480);
            StartPosition = FormStartPosition.CenterScreen;
            FormBorderStyle = FormBorderStyle.FixedSingle;
            MaximizeBox = false;
            DoubleBuffered = true;
            KeyPreview = true;

            // HUD
            hud.AutoSize = true;
            hud.Font = new Font(Font.FontFamily, 10, FontStyle.Bold);
            hud.Padding = new Padding(8);
            hud.Text = "Score: 0   [Strzałki] ruch  |  [Spacja] pauza/restart";
            Controls.Add(hud);

            // Canvas
            canvas.Location = new Point(0, hud.Height);
            canvas.Size = new Size(ClientSize.Width, ClientSize.Height - hud.Height);
            canvas.Anchor = AnchorStyles.Top | AnchorStyles.Bottom | AnchorStyles.Left | AnchorStyles.Right;
            canvas.BackColor = Color.Black;
            canvas.Paint += Canvas_Paint;
            Controls.Add(canvas);

            // Timer
            gameTimer.Interval = 120;
            gameTimer.Tick += GameTimer_Tick;

            // Klawiatura
            KeyDown += Form1_KeyDown;

            StartGame();
        }

        private void StartGame()
        {
            snake.Clear();
            score = 0;
            dir = Dir.Right;
            isGameOver = false;

            snake.Add(new Point(5, 5));
            snake.Add(new Point(4, 5));
            snake.Add(new Point(3, 5));

            SpawnFood();
            gameTimer.Start();
            UpdateHud();
            canvas.Invalidate();
        }

        private void UpdateHud()
        {
            hud.Text = $"Score: {score}   [Strzałki] ruch  |  [Spacja] pauza/restart";
        }

        private (int cols, int rows) GridSize()
        {
            int cols = Math.Max(1, canvas.ClientSize.Width / cell);
            int rows = Math.Max(1, canvas.ClientSize.Height / cell);
            return (cols, rows);
        }

        private void SpawnFood()
        {
            var (cols, rows) = GridSize();
            do
            {
                food = new Point(rng.Next(cols), rng.Next(rows));
            } while (snake.Contains(food));
        }

        // == Uwaga: object? sender ==
        private void GameTimer_Tick(object? sender, EventArgs e)
        {
            if (isGameOver) return;

            var head = snake.First();
            var next = head;

            switch (dir)
            {
                case Dir.Right: next.X++; break;
                case Dir.Left: next.X--; break;
                case Dir.Down: next.Y++; break;
                case Dir.Up: next.Y--; break;
            }

            var (cols, rows) = GridSize();

            bool hitWall = next.X < 0 || next.Y < 0 || next.X >= cols || next.Y >= rows;
            bool hitSelf = snake.Contains(next);

            if (hitWall || hitSelf)
            {
                gameTimer.Stop();
                isGameOver = true;
                canvas.Invalidate();
                return;
            }

            snake.Insert(0, next);

            if (next == food)
            {
                score += 10;
                UpdateHud();
                SpawnFood();
            }
            else
            {
                snake.RemoveAt(snake.Count - 1);
            }

            canvas.Invalidate();
        }

        private void Canvas_Paint(object? sender, PaintEventArgs e)
        {
            var g = e.Graphics;

            using (var foodBrush = new SolidBrush(Color.Red))
            {
                g.FillRectangle(foodBrush, food.X * cell, food.Y * cell, cell, cell);
            }

            for (int i = 0; i < snake.Count; i++)
            {
                var p = snake[i];
                var color = i == 0 ? Color.Lime : Color.Green;
                using (var b = new SolidBrush(color))
                {
                    g.FillRectangle(b, p.X * cell, p.Y * cell, cell, cell);
                }
            }

            if (isGameOver)
            {
                string msg = "GAME OVER — [Spacja] aby zagrać ponownie";
                using (var sf = new StringFormat { Alignment = StringAlignment.Center, LineAlignment = StringAlignment.Center })
                using (var brush = new SolidBrush(Color.White))
                using (var font = new Font(Font.FontFamily, 14, FontStyle.Bold))
                {
                    g.DrawString(msg, font, brush, canvas.ClientRectangle, sf);
                }
            }
        }

        private void Form1_KeyDown(object? sender, KeyEventArgs e)
        {
            if (e.KeyCode == Keys.Right && dir != Dir.Left) dir = Dir.Right;
            else if (e.KeyCode == Keys.Left && dir != Dir.Right) dir = Dir.Left;
            else if (e.KeyCode == Keys.Up && dir != Dir.Down) dir = Dir.Up;
            else if (e.KeyCode == Keys.Down && dir != Dir.Up) dir = Dir.Down;

            if (e.KeyCode == Keys.Space)
            {
                if (isGameOver) StartGame();
                else
                {
                    if (gameTimer.Enabled) gameTimer.Stop();
                    else gameTimer.Start();
                }
            }
        }

        // Ten handler może zostać w designerze – dostosowany do nullability:
        private void Form1_Load(object? sender, EventArgs e) { }
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
using System.Linq;
using System.Windows.Forms;

namespace QuizWinForms
{
    public partial class Form1 : Form
    {
        // Prosty model pytania
        private class Question
        {
            public string Text { get; set; }
            public string[] Options { get; set; } // 4 odpowiedzi
            public int CorrectIndex { get; set; } // 0..3
        }

        private List<Question> _quiz;
        private int _currentIndex = 0;
        private int _score = 0;
        private bool _answered = false;

        public Form1()
        {
            InitializeComponent();

            // Podpisy przycisków (możesz ustawić też w projektancie)
            button1.Text = "Sprawdź";
            button2.Text = "Dalej";

            // Zdarzenia
            button1.Click += ButtonCheck_Click;
            button2.Click += ButtonNext_Click;
            this.Load += Form1_Load;

            // Kliknięcie dowolnego radiobuttona odblokuje "Sprawdź"
            radioButton1.CheckedChanged += AnyOption_CheckedChanged;
            radioButton2.CheckedChanged += AnyOption_CheckedChanged;
            radioButton3.CheckedChanged += AnyOption_CheckedChanged;
            radioButton4.CheckedChanged += AnyOption_CheckedChanged;
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            // Zestaw przykładowych pytań
            _quiz = new List<Question>
            {
                new Question
                {
                    Text = "Stolica Polski to…",
                    Options = new[] {"Kraków", "Gdańsk", "Warszawa", "Wrocław"},
                    CorrectIndex = 2
                },
                new Question
                {
                    Text = "Ile to 3 × 7?",
                    Options = new[] {"20", "21", "24", "27"},
                    CorrectIndex = 1
                },
                new Question
                {
                    Text = "Który typ w C# jest typu referencyjnego?",
                    Options = new[] {"int", "double", "bool", "string"},
                    CorrectIndex = 3
                }
            };

            _currentIndex = 0;
            _score = 0;
            UpdateScoreLabel();
            LoadQuestion();
        }

        private void LoadQuestion()
        {
            _answered = false;
            EnableOptions(true);
            ClearSelection();
            button1.Enabled = false;       // „Sprawdź” aktywne dopiero po wyborze
            button2.Enabled = false;       // „Dalej” dopiero po sprawdzeniu
            label3.Text = "";              // czyść feedback

            var q = _quiz[_currentIndex];
            label1.Text = $"Pytanie {(_currentIndex + 1)}/{_quiz.Count}: {q.Text}";
            radioButton1.Text = q.Options[0];
            radioButton2.Text = q.Options[1];
            radioButton3.Text = q.Options[2];
            radioButton4.Text = q.Options[3];

            // Upewnij się, że nazwy RadioButtonów są widoczne (gdyby ktoś je zmienił)
            radioButton1.Visible = radioButton2.Visible = radioButton3.Visible = radioButton4.Visible = true;

            // Ustaw napis na przycisku „Dalej”/„Restart”
            button2.Text = (_currentIndex == _quiz.Count - 1) ? "Zakończ" : "Dalej";
        }

        private void ButtonCheck_Click(object sender, EventArgs e)
        {
            if (_answered) return;

            int selected = GetSelectedIndex();
            if (selected == -1) return;

            _answered = true;
            EnableOptions(false);      // zablokuj odpowiedzi po sprawdzeniu
            button2.Enabled = true;    // można przejść dalej

            var q = _quiz[_currentIndex];
            if (selected == q.CorrectIndex)
            {
                _score++;
                label3.Text = "✅ Dobrze!";
            }
            else
            {
                label3.Text = $"❌ Źle. Poprawna odpowiedź: {q.Options[q.CorrectIndex]}";
            }
            UpdateScoreLabel();
        }

        private void ButtonNext_Click(object sender, EventArgs e)
        {
            if (!_answered) return; // najpierw sprawdź

            if (_currentIndex < _quiz.Count - 1)
            {
                _currentIndex++;
                LoadQuestion();
            }
            else
            {
                // Koniec quizu – pokaż podsumowanie i opcję restartu
                MessageBox.Show($"Koniec! Twój wynik: {_score}/{_quiz.Count}", "Quiz");
                // Restart
                _currentIndex = 0;
                _score = 0;
                UpdateScoreLabel();
                LoadQuestion();
                label3.Text = "Nowa runda – powodzenia!";
            }
        }

        private void AnyOption_CheckedChanged(object sender, EventArgs e)
        {
            if (!_answered)
                button1.Enabled = GetSelectedIndex() != -1;
        }

        private void UpdateScoreLabel()
        {
            label2.Text = $"Wynik: {_score}/{_quiz.Count}";
        }

        private void EnableOptions(bool enabled)
        {
            radioButton1.Enabled = enabled;
            radioButton2.Enabled = enabled;
            radioButton3.Enabled = enabled;
            radioButton4.Enabled = enabled;
        }

        private void ClearSelection()
        {
            radioButton1.Checked = false;
            radioButton2.Checked = false;
            radioButton3.Checked = false;
            radioButton4.Checked = false;
        }

        private int GetSelectedIndex()
        {
            if (radioButton1.Checked) return 0;
            if (radioButton2.Checked) return 1;
            if (radioButton3.Checked) return 2;
            if (radioButton4.Checked) return 3;
            return -1;
        }
    }
}


```
