Up: [Readme.md](../Readme.md),  Prev: [Section 4](sec4.md), Next: [Section 6](sec6.md)

# Widgets (2)

## GtkTextView, GtkTextbuffer and GtkScrolledWindow

### GtkTextView and GtkTextBuffer

GtkTextview is a widget for multi-line text editing.
GtkTextBuffer is a text buffer which is connected to GtkTextView.
See a sample program `tfv1.c` below.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 on_activate (GApplication *app, gpointer user_data) {
 5   GtkWidget *win;
 6   GtkWidget *tv;
 7   GtkTextBuffer *tb;
 8   gchar *text;
 9 
10   text = 
11 "Once upon a time, there was an old man who was called Taketori-no-Okina."
12 "It is a japanese word that means a man whose work is making bamboo baskets.\n"
13 "One day, he went into a mountain and found a shining bamboo."
14 "\"What a mysterious bamboo it is!,\" he said."
15 "He cut it, then there was a small cute baby girl in it."
16 "The girl was shining faintly."
17 "He thought this baby girl is a gift from Heaven and took her home.\n"
18 "His wife was surprized at his tale."
19 "They were very happy because they had no children."
20 ;
21   win = gtk_application_window_new (GTK_APPLICATION (app));
22   gtk_window_set_title (GTK_WINDOW (win), "Taketori");
23   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
24 
25   tv = gtk_text_view_new ();
26   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
27   gtk_text_buffer_set_text (tb, text, -1);
28   gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
29 
30   gtk_window_set_child (GTK_WINDOW (win), tv);
31 
32   gtk_widget_show (win);
33 }
34 
35 int
36 main (int argc, char **argv) {
37   GtkApplication *app;
38   int stat;
39 
40   app = gtk_application_new ("com.github.ToshioCP.tfv1", G_APPLICATION_FLAGS_NONE);
41   g_signal_connect (app, "activate", G_CALLBACK (on_activate), NULL);
42   stat =g_application_run (G_APPLICATION (app), argc, argv);
43   g_object_unref (app);
44   return stat;
45 }
46 
~~~

Look at line 25.
GtkTextView is generated and its pointer is assigned to `tv`.
When GtkTextView is generated, the connected GtkTextBuffer is also generated automatically.
In the next line, the pointer to the buffer is got and assigned to `tb`.
Then, the text from line 10 to 20 is assigned to the buffer.

GtkTextView has a wrap mode.
When it is set to `GTK_WRAP_WORD_CHAR`, text wraps in between words, or if that is not enough, also between graphemes.

In line 30, `tv` is added to `win` as a child.

Now compile and run it.

![GtkTextView](../image/screenshot_tfv1.png)

There's an I-beam pointer in the window.
You can add or delete any characters on GtkTextview.
And your change is kept in GtkTextBuffer.
If you add more characters than the limit of the window, the height of the window extends.
If the height gets bigger than the height of the display screen, you won't be able to control the size of the window back to the original size.
It's a problem and it shows that there exists a bug in the program.
You can solve it by putting GtkScrolledWindow between GtkApplicationWindow and GtkTextView.

### GtkScrolledWindow

What we need to do is:

- Generate GtkScrolledWindow and insert it to GtkApplicationWindow as a child.
- insert GtkTextView to GtkScrolledWindow as a child.

Modify `tfv1.c` and save it as `tfv2.c`.
The difference between these two files is very little.

~~~
$ cd tfv; diff tfv1.c tfv2.c
5a6
>   GtkWidget *scr;
24a26,28
>   scr = gtk_scrolled_window_new ();
>   gtk_window_set_child (GTK_WINDOW (win), scr);
> 
30c34
<   gtk_window_set_child (GTK_WINDOW (win), tv);
---
>   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
40c44
<   app = gtk_application_new ("com.github.ToshioCP.tfv1", G_APPLICATION_FLAGS_NONE);
---
>   app = gtk_application_new ("com.github.ToshioCP.tfv2", G_APPLICATION_FLAGS_NONE);
~~~

Though you can modify the source file by this diff output, It's good for you to show `tfv2.c`.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 on_activate (GApplication *app, gpointer user_data) {
 5   GtkWidget *win;
 6   GtkWidget *scr;
 7   GtkWidget *tv;
 8   GtkTextBuffer *tb;
 9   gchar *text;
10 
11   text = 
12 "Once upon a time, there was an old man who was called Taketori-no-Okina."
13 "It is a japanese word that means a man whose work is making bamboo baskets.\n"
14 "One day, he went into a mountain and found a shining bamboo."
15 "\"What a mysterious bamboo it is!,\" he said."
16 "He cut it, then there was a small cute baby girl in it."
17 "The girl was shining faintly."
18 "He thought this baby girl is a gift from Heaven and took her home.\n"
19 "His wife was surprized at his tale."
20 "They were very happy because they had no children."
21 ;
22   win = gtk_application_window_new (GTK_APPLICATION (app));
23   gtk_window_set_title (GTK_WINDOW (win), "Taketori");
24   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
25 
26   scr = gtk_scrolled_window_new ();
27   gtk_window_set_child (GTK_WINDOW (win), scr);
28 
29   tv = gtk_text_view_new ();
30   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
31   gtk_text_buffer_set_text (tb, text, -1);
32   gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
33 
34   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
35 
36   gtk_widget_show (win);
37 }
38 
39 int
40 main (int argc, char **argv) {
41   GtkApplication *app;
42   int stat;
43 
44   app = gtk_application_new ("com.github.ToshioCP.tfv2", G_APPLICATION_FLAGS_NONE);
45   g_signal_connect (app, "activate", G_CALLBACK (on_activate), NULL);
46   stat =g_application_run (G_APPLICATION (app), argc, argv);
47   g_object_unref (app);
48   return stat;
49 }
50 
~~~

Now compile and run it.
This time the window doesn't extend even if you type a lot of characters.
It just scrolls.


Up: [Readme.md](../Readme.md),  Prev: [Section 4](sec4.md), Next: [Section 6](sec6.md)
