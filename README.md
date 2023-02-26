Rather than endure the "awkwardness" you describe in your comment, one option would be to implement your `captureFromScreen()` method by _really_ capturing the screen using `Graphics.CopyFromScreen()`. This way it should be *WYSIWYG*.

    PrintPreviewDialog _printPreview = new PrintPreviewDialog();
    private void captureFromScreen()
    {
        GraphicsUnit unit = GraphicsUnit.Pixel;
        Bitmap bitmap = new Bitmap(panel.Width, panel.Height);
        var screenLocation = PointToScreen(panel.Location);
        using (var graphics = Graphics.FromImage(bitmap))
        {
            graphics.CopyFromScreen(screenLocation, Point.Empty, panel.Size);
        }

        PrintDocument document = new PrintDocument();
        document.PrintPage += localPrintPage;
        _printPreview.Document= document;

        _printPreview.ShowDialog();

        void localPrintPage(object sender, PrintPageEventArgs e)
        {
            e.Graphics?.DrawImage(
                bitmap,
                    (e.PageBounds.Width - bitmap.Width) / 2,
                    (e.PageBounds.Height - bitmap.Height) / 2,
                    bitmap.Width,
                    bitmap.Height);
        }
    }

***
**Test**

[![print preview][1]][1]

    public partial class MainForm : Form, IMessageFilter
    {
        public MainForm()
        {
            InitializeComponent();
            Application.AddMessageFilter(this);
            Disposed += (sender, e) => Application.RemoveMessageFilter(this);

            Label label = new Label
            {
                Name = "label1",
                Text = "Label",
                Location = new Point(0, panel.Height/2),
                Size = new Size(panel.Width, 80),
                BackColor= Color.FromArgb(231, 134, 131),
                TextAlign = ContentAlignment.MiddleCenter,
            };
            panel.BackColor = Color.Azure;
            panel.Controls.Add(label);
            panel.Controls.SetChildIndex(label, 0);
        }
        const int WM_KEYDOWN = 0x0100;
        public bool PreFilterMessage(ref Message m)
        {
            if(m.Msg.Equals(WM_KEYDOWN))
            {
                switch((Keys)m.WParam | ModifierKeys) 
                {
                    case Keys.Control | Keys.P:
                        BeginInvoke(new Action(()=> captureFromScreen()));
                        return true;
                }
            }
            return false;
        }
        .
        .
        .
    }


  [1]: https://i.stack.imgur.com/waGLb.png