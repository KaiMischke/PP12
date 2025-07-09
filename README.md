# PP12: GUI Programming with X11 and GTK+

## Goal

In this exercise you will:

1. Work directly with the X11 (Xlib) API to create a window and draw primitive graphics.
2. Install GTK+ and build a simple GTK3 application, then extend it with an additional feature.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. When time is up, stop immediately and record where you paused.

---

## Workflow

1. **Fork** this repository on GitHub.
2. **Clone** your fork locally.
3. Create a `solutions/` directory at the project root:

   ```bash
   mkdir solutions
   ```
4. Add each task’s source file(s) under `solutions/`.
5. **Commit** and **push** your solutions.
6. **Submit** your GitHub repo link for review.

---

## Prerequisites

* X11 development libraries (`libx11-dev`).
* GTK+ 3 development libraries (`libgtk-3-dev`).
* GNU C compiler (`gcc`).

---

## Tasks

### Task 1: Bare X11 Window & Drawing

**Objective:** Use Xlib directly to open a window and draw a rectangle.

1. Create `solutions/x11_draw.c` with the following skeleton:

   ```c
   #include <X11/Xlib.h>
   #include <stdlib.h>
   #include <stdio.h>

   int main(void) {
       Display *dpy = XOpenDisplay(NULL);
       if (!dpy) {
           fprintf(stderr, "Cannot open display\n");
           return EXIT_FAILURE;
       }
       int screen = DefaultScreen(dpy);
       Window win = XCreateSimpleWindow(
           dpy, RootWindow(dpy, screen),
           10, 10, 400, 300, 1,
           BlackPixel(dpy, screen),
           WhitePixel(dpy, screen)
       );
       XSelectInput(dpy, win, ExposureMask | KeyPressMask);
       XMapWindow(dpy, win);

       GC gc = XCreateGC(dpy, win, 0, NULL);
       XSetForeground(dpy, gc, BlackPixel(dpy, screen));

       for(;;) {
           XEvent e;
           XNextEvent(dpy, &e);
           if (e.type == Expose) {
               // Draw a rectangle
               XDrawRectangle(dpy, win, gc, 50, 50, 200, 100);
           }
           if (e.type == KeyPress)
               break;
       }

       XFreeGC(dpy, gc);
       XDestroyWindow(dpy, win);
       XCloseDisplay(dpy);
       return EXIT_SUCCESS;
   }
   ```
2. Compile and run:

   ```bash
   gcc -o solutions/x11_draw solutions/x11_draw.c -lX11
   ./solutions/x11_draw
   ```

#### Reflection Questions

1. **What steps are required to open an X11 window and receive events?**

**Lösung zu Aufgabe 1:**
Um ein X11-Fenster zu öffnen und Events zu empfangen, sind diese Schritte nötig:

- *Display-Verbindung öffnen*:
  Mit `XOpenDisplay(NULL)` eine Verbindung zum X-Server herstellen (meistens der lokale X-Server).

- *Screen auswählen:*
  Mit `DefaultScreen(dpy)` den Standardbildschirm bestimmen.

- *Fenster erstellen:*
  Mit `XCreateSimpleWindow()` oder `XCreateWindow()` ein neues Fenster auf dem Root-Window des Bildschirms anlegen. Dabei Position, Größe, Rahmenstärke und Farben festlegen.

- *Events erhalten:*
  Mit `XSelectInput()` den Event-Masken (z. B. `ExposureMask`, `KeyPressMask`) festlegen, welche Ereignisse das Fenster erhalten soll.

- *Fenster anzeigen:*
  Mit `XMapWindow()`das Fenster sichtbar machen.

- *Grafik-Kontext erstellen:*
  Mit `XCreateGC()` einen Grafik-Kontext für das Fenster anlegen, um später zeichnen zu können.

- *Event-Schleife starten:*
  Mit einer Endlosschleife `XNextEvent()` auf neue Events warten und diese abarbeiten.
  
2. **How does the `Expose` event trigger your drawing code?**

**Lösung zu Aufgabe 2:**

- Der **Expose-Event** signalisiert, dass ein Teil (oder das ganze Fenster) neu gezeichnet werden muss, z. B. weil das Fenster gerade sichtbar geworden ist oder überdeckt war.
- Sobald `XNextEvent()` ein Event vom Typ `Expose` liefert, führt das Programm im Event-Handler den Zeichen-Code aus (z. B. `XDrawRectangle()`), um die Grafik im Fenster zu aktualisieren.
- Ohne diese Reaktion auf Expose-Events würde das Fenster nach dem Öffnen oder Wiederfreigeben leer oder unvollständig bleiben.

---

### Task 2: GTK+ 3 Application & Extension

**Objective:** Install GTK+ 3, build a basic GTK application, then add a new interactive feature.

1. Install GTK+ 3:

   ```bash
   sudo apt update && sudo apt install libgtk-3-dev
   ```
2. Create `solutions/gtk_app.c` with this skeleton:

   ```c
   #include <gtk/gtk.h>

   static void on_button_clicked(GtkButton *button, gpointer user_data) {
       GtkLabel *label = GTK_LABEL(user_data);
       gtk_label_set_text(label, "Button clicked!");
   }

   int main(int argc, char **argv) {
       gtk_init(&argc, &argv);

       GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL);
       gtk_window_set_title(GTK_WINDOW(window), "GTK+ Demo");
       gtk_window_set_default_size(GTK_WINDOW(window), 300, 200);
       g_signal_connect(window, "destroy", G_CALLBACK(gtk_main_quit), NULL);

       GtkWidget *vbox = gtk_box_new(GTK_ORIENTATION_VERTICAL, 5);
       gtk_container_add(GTK_CONTAINER(window), vbox);

       GtkWidget *label = gtk_label_new("Hello, GTK+!");
       gtk_box_pack_start(GTK_BOX(vbox), label, TRUE, TRUE, 0);

       GtkWidget *button = gtk_button_new_with_label("Click me");
       g_signal_connect(button, "clicked",
                        G_CALLBACK(on_button_clicked), label);
       gtk_box_pack_start(GTK_BOX(vbox), button, TRUE, TRUE, 0);

       gtk_widget_show_all(window);
       gtk_main();
       return 0;
   }
   ```
3. Compile and run:

   ```bash
   gcc -o solutions/gtk_app solutions/gtk_app.c $(pkg-config --cflags --libs gtk+-3.0)
   ./solutions/gtk_app
   ```
4. **Extension Feature:** Modify the program to add a **text entry** (`GtkEntry`) above the button, and on button click update the label to show the entry’s current text instead of the fixed string.

   * Hint: use `gtk_entry_get_text()`.

#### Reflection Questions

1. **How does GTK’s signal-and-callback mechanism differ from X11’s event loop?**

**Lösung zu Aufgabe 1:**

**X11:**
- Arbeitet mit einem **low-level Event-Loop**, der direkt vom Programm implementiert wird.
- Man ruft `XNextEvent()` in einer Schleife auf, um Events vom X-Server abzuholen.
- Dann wird der Event-Typ geprüft (z. B. `Expose`, `KeyPress`) und der entsprechende Code ausgeführt.
- Die Steuerung liegt komplett beim Programmierer.
  
 **GTK:**
 - Baut auf X11 (oder anderen Window-Systemen) auf, abstrahiert das Event-Handling.
 - Verwendet ein **Signal- und Callback-System**:
 - Widgets senden Signale (z. B. „clicked“, „destroy“).
 - Man verbindet (connectet) Callback-Funktionen mit diesen Signalen.
 - Das GTK-Framework verwaltet intern die Event-Schleife und ruft die Callback-Funktionen auf, wenn das Signal        ausgelöst wird.
 - Bietet eine höhere Abstraktion und mehr Komfort, da man sich nicht um Details des Event-Pollings kümmern musst.

2. **Why use `pkg-config` when compiling GTK applications?**

**Lösung zu Aufgabe 2:**

- GTK+ ist eine umfangreiche Bibliothek mit vielen Abhängigkeiten und spezifischen Compiler- und Linker-Flags (z. B. Include-Pfade, Linker-Optionen).
- `pkg-config` stellt sicher, dass man:
- Alle notwendigen **Compiler-Optionen** (`--cflags`) bekommt (z. B. Pfade zu Header-Dateien).
- Alle notwendigen **Linker-Optionen** (`--libs`) bekommt (z. B. GTK-Bibliotheken, Abhängigkeiten).
- Dadurch werden Fehler beim Kompilieren und Linken vermieden.
- Außerdem sorgt es für **Portabilität**: Egal, wo GTK auf dem System installiert ist, `pkg-config` findet die richtigen Einstellungen.

---

**Remember:** Stop after **90 minutes** and record where you stopped.
