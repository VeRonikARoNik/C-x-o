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
