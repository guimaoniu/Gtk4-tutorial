Up: [Readme.md](../Readme.md),  Prev: [Section 11](sec11.md), Next: [Section 13](sec13.md)

# Functions in TfeTextView

In this section I will explain functions in TfeTextView object.

### tfe.h and tfetextview.h

`tfe.h` is a top header file and it includes `gtk.h` and all the header files.
C source files `tfeapplication.c` and `tfenotebook.c` include `tfe.h` at the beginning.

~~~C
1 #include <gtk/gtk.h>
2 
3 #include "../tfetextview/tfetextview.h"
4 #include "tfenotebook.h"
~~~

`../tfetextview/tfetextview.h` is a header file which describes the public functions in `tfetextview.c`.

~~~C
 1 #ifndef __TFE_TEXT_VIEW_H__
 2 #define __TFE_TEXT_VIEW_H__
 3 
 4 #include <gtk/gtk.h>
 5 
 6 #define TFE_TYPE_TEXT_VIEW tfe_text_view_get_type ()
 7 G_DECLARE_FINAL_TYPE (TfeTextView, tfe_text_view, TFE, TEXT_VIEW, GtkTextView)
 8 
 9 /* "open-response" signal response */
10 enum TfeTextViewOpenResponseType
11 {
12   TFE_OPEN_RESPONSE_SUCCESS,
13   TFE_OPEN_RESPONSE_CANCEL,
14   TFE_OPEN_RESPONSE_ERROR
15 };
16 
17 GFile *
18 tfe_text_view_get_file (TfeTextView *tv);
19 
20 void
21 tfe_text_view_open (TfeTextView *tv, GtkWidget *win);
22 
23 void
24 tfe_text_view_save (TfeTextView *tv);
25 
26 void
27 tfe_text_view_saveas (TfeTextView *tv);
28 
29 GtkWidget *
30 tfe_text_view_new_with_file (GFile *file);
31 
32 GtkWidget *
33 tfe_text_view_new (void);
34 
35 #endif /* __TFE_TEXT_VIEW_H__ */
~~~

- 1,2,35: Thanks to these three lines, the following lines are included only once. 
- 4: Includes gtk4 header files.
The header file `gtk4` also has the same mechanism to avoid including it multiple times.
- 6-7: These two lines define TfeTextView.
- 9-15: A definition of the value of the parameter of "open-response" signal.
- 17-33: Declaration of public functions on GtkTextView.

## Functions to generate TfeTextView object

TfeTextView Object is generated by `tfe_text_view_new` or `tfe_text_view_new_with_file`.

~~~C
GtkWidget *tfe_text_view_new (void);
~~~

`tfe_text_view_new` just generates a new TfeTextView object and returns the pointer to the new object.

~~~C
GtkWidget *tfe_text_view_new_with_file (GFile *file);
~~~

`tfe_text_view_new_with_file` is given a Gfile object as the argument and it loads the file into the GtkTextBuffer object, then returns the pointer to the new object.
If an error occurs during the generation process, NULL is returned.

Each function is defined as follows.

~~~C
 1 GtkWidget *
 2 tfe_text_view_new_with_file (GFile *file) {
 3   g_return_val_if_fail (G_IS_FILE (file), NULL);
 4 
 5   GtkWidget *tv;
 6   GtkTextBuffer *tb;
 7   char *contents;
 8   gsize length;
 9 
10   if (! g_file_load_contents (file, NULL, &contents, &length, NULL, NULL)) /* read error */
11     return NULL;
12 
13   tv = tfe_text_view_new();
14   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
15   gtk_text_buffer_set_text (tb, contents, length);
16   g_free (contents);
17   TFE_TEXT_VIEW (tv)->file = g_file_dup (file);
18   return tv;
19 }
20 
21 GtkWidget *
22 tfe_text_view_new (void) {
23   return GTK_WIDGET (g_object_new (TFE_TYPE_TEXT_VIEW, NULL));
24 }
~~~

- 21-24: `tfe_text_view_new` function.
Just returns the value from the function `g_object_new` but casts it to the pointer to GtkWidget.
Initialization is done in `tfe_text_view_init` which is called in the process of `g_object_new` function.
- 1-19: `tfe_text_view_new_with_file` function.
- 3: `g_return_val_if_fail` is described in [Glib API reference](https://developer.gnome.org/glib/stable/glib-Warnings-and-Assertions.html#g-return-val-if-fail).
It tests whether the argument `file` is a pointer to GFile.
If it's true, then the program goes on to the next line.
If it's false, then it returns NULL (the second argument) immediately.
And at the same time it logs out the error message (usually the log is outputted to stderr or stdout).
This function is used to check the programmer's error.
If an error occurs, the solution is usually to change the (caller) program and fix the bug.
You need to distinguish programmer's errors and runtime errors.
You shouldn't use this function to find runtime errors.
- 10-11: If an error occurs when reading the file, then return NULL.
- 13: Calls the function `tfe_text_view_new`.
The function generates TfeTextView instance and returns the pointer to the instance.
- 14: Gets the pointer to GtkTextBuffer corresponds to `tv`.
The pointer is assigned to `tb`
- 15: Assigns the contents read from the file to GtkTextBuffer pointed by `tb`.
- 16: Frees the memories pointed by `contents`.
- 17: Duplicates `file` and sets `tv->file` to point it.
- 18: Returns `tv`, which is a pointer to the newly created TfeTextView instance..

## Save and saveas functions

Save and saveas functions write the contents in GtkTextBuffer to a file.

~~~C
void tfe_text_view_save (TfeTextView *tv)
~~~

The function `save` writes the contents in GtkTextBuffer to a file specified by `tv->file`.
If `tv->file` is NULL, then it shows GtkFileChooserDialog and prompts the user to choose a file to save.
Then it saves the contents to the file and sets `tv->file` to point the GFile instance of the file.

~~~C
void tfe_text_view_saveas (TfeTextView *tv)
~~~

The function `saveas` uses GtkFileChooserDialog and prompts the user to select a existed file or specify a new file to save.
Then, the function changes `tv->file` and save the contents to the specified file.
If an error occurs, it is shown to the user through the message dialog.
The error is managed only in the TfeTextView instance and no information is notified to the caller.

~~~C
 1 static void
 2 saveas_dialog_response (GtkWidget *dialog, gint response, TfeTextView *tv) {
 3   GtkTextBuffer *tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
 4   GFile *file;
 5   GtkWidget *win = gtk_widget_get_ancestor (GTK_WIDGET (tv), GTK_TYPE_WINDOW);
 6 
 7   gtk_window_destroy (GTK_WINDOW (dialog));
 8   if (response == GTK_RESPONSE_ACCEPT) {
 9     file = gtk_file_chooser_get_file (GTK_FILE_CHOOSER (dialog));
10     if (! G_IS_FILE (file))
11       g_warning ("TfeTextView: gtk_file_chooser_get_file returns non GFile object.\n");
12     else {
13       save_file(file, tb, GTK_WINDOW (win));
14       if (G_IS_FILE (tv->file))
15         g_object_unref (tv->file);
16       tv->file = file;
17       gtk_text_buffer_set_modified (tb, FALSE);
18       g_signal_emit (tv, tfe_text_view_signals[CHANGE_FILE], 0);
19     }
20   }
21 }
22 
23 void
24 tfe_text_view_save (TfeTextView *tv) {
25   g_return_if_fail (TFE_IS_TEXT_VIEW (tv));
26 
27   GtkTextBuffer *tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
28   GtkWidget *win = gtk_widget_get_ancestor (GTK_WIDGET (tv), GTK_TYPE_WINDOW);
29 
30   if (! gtk_text_buffer_get_modified (tb))
31     return; /* no need to save it */
32   else if (tv->file == NULL)
33     tfe_text_view_saveas (tv);
34   else if (! G_IS_FILE (tv->file))
35     g_error ("TfeTextView: The pointer in this object isn't NULL nor GFile object.\n");
36   else {
37     if (save_file (tv->file, tb, GTK_WINDOW (win)))
38       gtk_text_buffer_set_modified (tb, FALSE);
39   }
40 }
41 
42 void
43 tfe_text_view_saveas (TfeTextView *tv) {
44   g_return_if_fail (TFE_IS_TEXT_VIEW (tv));
45 
46   GtkWidget *dialog;
47   GtkWidget *win = gtk_widget_get_ancestor (GTK_WIDGET (tv), GTK_TYPE_WINDOW);
48 
49   dialog = gtk_file_chooser_dialog_new ("Save file", GTK_WINDOW (win), GTK_FILE_CHOOSER_ACTION_SAVE,
50                                       "Cancel", GTK_RESPONSE_CANCEL,
51                                       "Save", GTK_RESPONSE_ACCEPT,
52                                       NULL);
53   g_signal_connect (dialog, "response", G_CALLBACK (saveas_dialog_response), tv);
54   gtk_widget_show (dialog);
55 }
~~~

- 20-56: `Tfe_text_view_save` function.
- 22: If `tv` is not a pointer to TfeTextView, then it logs an error message and immediately returns.
This function is similar to `g_return_val_if_fail` function, but no value is returned because `tfe_text_view_save` doesn't return a value.
- 32-33: If the buffer hasn't modified, then it doesn't need to save it.
So the function returns.
- 34-35: If `tv->file` is NULL, no file has given yet.
It calls `tfe_text_view_saveas` which prompts a user to select a file or specify a new file to save.
- 37-38: Gets the contents of the GtkTextBuffer and sets `contents` to point it.
- 39-40: Saves the content to the file.
If it succeeds, it resets the modified bit in the GtkTextBuffer.
- 42-53: If file writing fails, it assigns NULL to `tv->file`.
Emits "change-file" signal.
Shows the error message dialog (48-52).
Because the handler is `gtk_window_destroy`, the dialog disappears when user clicks on the button in the dialog.
Frees the GError object.
- 58-71: `tfe_text_view_saveas` function.
It shows GtkFileChooserDialog and prompts the user to choose a file.
- 65-68: Generates GtkFileChooserDialog.
The title is "Save file".
Transient parent of the dialog is `win`, which is the top level window.
The action is save mode.
The buttons are Cancel and Save.
- 69: connects the "response" signal of the dialog and `saveas_dialog_response` handler.
- 1-18: `saveas_dialog_response` signal handler.
- 6-16: If the response is `GTK_RESPONSE_ACCEPT`, then it gets a pointer to the GFile object.
Then, it sets `tv->file` to point the GFile.
And turns on the modified bit of the GtkTextBuffer, emits "change-file" signal.
Finally, it calls `tfe_text_view_save` to save the buffer to the file.

![Saveas process](../image/saveas.png)

When you use GtkFileChooserDialog, you need to divide the program into two parts.
One is are a function which generates GtkFileChooserDialog and the other is a signal handler.
The function just generates and shows GtkFileChooserDialog.
The rest is done by the handler.
It gets Gfile from GtkFileChooserDialog and saves the buffer to the file by calling `tfe_text_view_save`.

## Open function

Open function shows GtkFileChooserDialog to users and prompts them to choose a file.
Then it reads the file and puts the text to GtkTextBuffer.

~~~C
void tfe_text_view_open (TfeTextView *tv, GtkWidget *win);
~~~

The parameter `win` is the top window.
It will be a transient parent window of GtkFileChooserDialog when the dialog is generated..
This allows window managers to keep the dialog on top of the parent window, or center the dialog over the parent window.
It is possible to give no parent window to the dialog.
However, it is encouraged to give a parent window to dialog.
This function might be called just after `tv` has been generated.
In that case, `tv` has not been incorporated into the widget hierarchy.
Therefore it is impossible to get the top window from `tv`.
That's why the function needs `win` parameter.

This function is usually called when the buffer of `tv` is empty.
However, even if the buffer is not empty, `tfe_text_view_open` doesn't treat it as an error.
If you want to revert the buffer, calling this function is appropriate.
Otherwise probably bad things will happen.

~~~C
 1 static void
 2 open_dialog_response(GtkWidget *dialog, gint response, TfeTextView *tv) {
 3   GtkTextBuffer *tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
 4   GFile *file;
 5   char *contents;
 6   gsize length;
 7   GtkWidget *message_dialog;
 8   GError *err = NULL;
 9 
10   if (response != GTK_RESPONSE_ACCEPT)
11     g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_CANCEL);
12   else if (! G_IS_FILE (file = gtk_file_chooser_get_file (GTK_FILE_CHOOSER (dialog)))) {
13     g_warning ("TfeTextView: gtk_file_chooser_get_file returns non GFile object.\n");
14     g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_ERROR);
15   } else if (! g_file_load_contents (file, NULL, &contents, &length, NULL, &err)) { /* read error */
16     if (G_IS_FILE (file))
17       g_object_unref (file);
18     message_dialog = gtk_message_dialog_new (GTK_WINDOW (dialog), GTK_DIALOG_MODAL,
19                                              GTK_MESSAGE_ERROR, GTK_BUTTONS_CLOSE,
20                                             "%s.\n", err->message);
21     g_signal_connect (message_dialog, "response", G_CALLBACK (gtk_window_destroy), NULL);
22     gtk_widget_show (message_dialog);
23     g_error_free (err);
24     g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_ERROR);
25   } else {
26     gtk_text_buffer_set_text (tb, contents, length);
27     g_free (contents);
28     if (G_IS_FILE (tv->file))
29       g_object_unref (tv->file);
30     tv->file = file;
31     gtk_text_buffer_set_modified (tb, FALSE);
32     g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_SUCCESS);
33     g_signal_emit (tv, tfe_text_view_signals[CHANGE_FILE], 0);
34   }
35   gtk_window_destroy (GTK_WINDOW (dialog));
36 }
37 
38 void
39 tfe_text_view_open (TfeTextView *tv, GtkWidget *win) {
40   g_return_if_fail (TFE_IS_TEXT_VIEW (tv));
41   g_return_if_fail (GTK_IS_WINDOW (win));
42 
43   GtkWidget *dialog;
44 
45   dialog = gtk_file_chooser_dialog_new ("Open file", GTK_WINDOW (win), GTK_FILE_CHOOSER_ACTION_OPEN,
46                                         "Cancel", GTK_RESPONSE_CANCEL,
47                                         "Open", GTK_RESPONSE_ACCEPT,
48                                         NULL);
49   g_signal_connect (dialog, "response", G_CALLBACK (open_dialog_response), tv);
50   gtk_widget_show (dialog);
51 }
~~~

- 37-50: `tfe_text_view_open` function.
- 44-47: Generates GtkFileChooserDialog.
The title is "Open file".
Transient parent window is the top window of the application, which is given by the caller.
The action is open mode.
The buttons are Cancel and Open.
- 48: connects the "response" signal of the dialog and `open_dialog_response` signal handler.
- 49: Shows the dialog.
- 1-35: `open_dialog_response` signal handler.
- 10-11: If the response from GtkFileChooserDialog is not `GTK_RESPONSE_ACCEPT`, which means the user has clicked on the "Cancel" button or close button, then it emits "open-response" signal with the parameter `TFE_OPEN_RESPONSE_CANCEL`.
- 12-13: Gets a pointer to Gfile by `gtk_file_chooser_get_file`.
If it is not GFile, maybe an error occurred.
Then it emits "open-response" signal with the parameter `TFE_OPEN_RESPONSE_ERROR`.
- 14-23: If an error occurs at file reading, then it decreases the reference count of the Gfile, shows a message dialog to report the error to the user and emits "open-response" signal with the parameter `TFE_OPEN_RESPONSE_ERROR`.
- 24-33: If the file has successfully read, then the text is inserted to GtkTextBuffer, frees the temporary buffer pointed by `contents` and sets `tv->file` to point the file (no duplication or unref is not necessary).
Then, it emits "open-response" signal with the parameter `TFE_OPEN_RESPONSE_SUCCESS` and emits "change-file" signal.
- 34: closes GtkFileCooserDialog.

Now let's think about the whole process between the other object (caller) and TfeTextView.
It is shown in the following diagram and you would think that it is really complicated.
Because signal is the only way for GtkFileChooserDialog to communicate with others.
In Gtk3, `gtk_dialog_run` function is available.
It simplifies the process.
However, in Gtk4, `gtk_dialog_run`is unavailable any more.

![Caller and TfeTextView](../image/open.png)

1. A caller gets a pointer `tv` to TfeTextView by calling `tfe_text_view_new`.
2. The caller connects the handler (left bottom in the diagram) and the signal "open-response".
3. It calls `tfe_text_view_open` to prompt the user to select a file from GtkFileChooserDialog.
4. The dialog emits a signal and it invokes the handler `open_dialog_response`.
5. The handler reads the file and inserts the text into GtkTextBuffer and emits a signal to inform the response status.
6. The handler outside TfeTextView receives the signal.

## Get file function

`gtk_text_view_get_file` is a simple function show as follows.

~~~C
1 GFile *
2 tfe_text_view_get_file (TfeTextView *tv) {
3   g_return_val_if_fail (TFE_IS_TEXT_VIEW (tv), NULL);
4 
5   if (G_IS_FILE (tv->file))
6     return g_file_dup (tv->file);
7   else
8     return NULL;
9 }
~~~

The important thing is to duplicate `tv->file`.
Otherwise, if the caller frees the GFile object, `tv->file` is no more guaranteed to point the GFile.

## API document and source file of tfetextview.c

Refer [API document of TfeTextView](../src/tfetextview/tfetextview_doc.md).
It is under the directory `src/tfetextview`.

All the source files are listed in [Section 15](sec15.md).
You can find them under [src/tfe5](../src/tfe5) and [src/tfetextview](../tfetextview) directories.


Up: [Readme.md](../Readme.md),  Prev: [Section 11](sec11.md), Next: [Section 13](sec13.md)
