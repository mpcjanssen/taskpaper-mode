

# Emacs TaskPaper Mode

TaskPaper mode is an Emacs major mode for working with files in TaskPaper format. The format was invented by Jesse Grosjean and named after his [TaskPaper][taskpaper] macOS app, which is a system for organizing your outlines and tasks in a text file. The format itself is exceptionally readable and supports different item types, outline hierarchy, and tagging.

TaskPaper format knows about four things: _projects_, _tasks_, _notes_, and _tags_. Items can be indented (using literal tabs) under other items to create outline structure, which defines parent-child hierarchical relationship:

    To create items:
        - To create a project, type a line ending with a colon.
        - To create a task, type a dash followed by a space.
        - To create a note, type any line that isn't a project or task.
        - To create a tag, type "@" followed by the tag's name.
            Tags can have a value in parentheses after the name.

The file format is fairly simple:

 - Files are expected to use the UTF-8 encoding and use `\n` to separate lines.

 - A task is a line that begins with a hyphen (`-`) followed by a space, which can optionally be prefixed (i.e. indented) with tabs. A task can have zero or more tags anywhere on the line (not just trailing at the end).

 - A project is a line that isn't a task and ends with a colon (`:`) followed by a newline. Tags can exist after the colon, but if any non-tag text is present, then it won't be recognized as a project.

 - A note is any non-blank line that doesn't match the task or project rules. A note can have zero or more tags anywhere on the line.

 - A tag consists of an at symbol (`@`) preceded by a space and followed by a tag name. Tags can optionally have a value in parentheses after the tag name.

Indentation level (with tabs, not spaces) defines ownership. For instance, if you indent one task under another task, then it is considered a subtask. Projects, tasks, and notes own all items that are indented underneath them. Empty lines are ignored when calculating ownership.

The system doesn't force any particular system on you; it provides basic list making elements for you to use as you see fit. See the [TaskPaper User's Guide][taskpaper-guide] for more details.

![](./images/screencast_01.gif)

![](./images/screencast_02.gif)

TaskPaper mode is implemented on top of Outline mode. Visibility cycling and structure editing help to work with the outline structure. Special commands also provided for outline filtering, tags manipulation, sorting, refiling, and archiving of items. For querying a collection of TaskPaper files, TaskPaper mode also includes a powerful agenda mode.

Documentation for TaskPaper mode is available below, but you can also use Emacs' help commands to access the built-in documentation. Documentation strings to each function are available via `C-h f` (`describe-function`), individual key bindings can be investigated with `C-h k` (`describe-key`), and a complete list of key bindings is available using `C-h m` (`describe-mode`).

This document explains the installation, usage, and basic customization of TaskPaper mode package. For more advanced customization, hacking and scripting see the [Scripting Guide][tp-mode-scripting-guide].


# Table of Contents

 - [Installation and Activation](#installation-and-activation)
 - [Usage](#usage)
    - [Formatting Items](#formatting-items)
    - [Folding](#folding)
    - [Outline Navigation](#outline-navigation)
    - [Structure Editing](#structure-editing)
    - [Tagging](#tagging)
    - [Completing Tasks](#completing-tasks)
    - [Calendar Integration](#calendar-integration)
    - [Date and Time Formats](#date-and-time-formats)
    - [Hyperlinks and Inline Images](#hyperlinks-and-inline-images)
    - [Sorting](#sorting)
    - [Filtering](#filtering)
    - [Searching](#searching)
    - [Refiling](#refiling)
    - [Archiving](#archiving)
    - [Multi-Document Support and Agenda View](#multi-document-support-and-agenda-view)
    - [Miscellaneous](#miscellaneous)
 - [Customization](#customization)
    - [Syntax Highlighting](#syntax-highlighting)
    - [Cleaner Outline View](#cleaner-outline-view)
    - [Indentation Guides](#indentation-guides)
 - [Acknowledgments](#acknowledgements)
 - [Bugs](#bugs)
 - [License](#license)


# Installation and Activation

Put `taskpaper-mode.el` on the load path and copy the following into your init file:

    (require 'taskpaper-mode)

Files with the `.taskpaper` extension use TaskPaper mode by default. If you want other filenames to be associated with TaskPaper mode, add the corresponding entry to the `auto-mode-alist` in your init file:

    (add-to-list 'auto-mode-alist
                 '("\\.todo\\'" . taskpaper-mode))


# Usage


## Formatting Items


In TaskPaper mode each line makes a new item.

 - `RET`: Create a new item with the same level as the one under cursor (`taskpaper-new-item-same-level`).

 - `M-RET`: Create a new task with the same level as the item under cursor (`taskpaper-new-task-same-level`).

The following commands reformat the current item.

 - `C-c C-f p`: Format item under cursor as project (`taskpaper-item-format-as-project`).

 - `C-c C-f t`: Format item under cursor as task (`taskpaper-item-format-as-task`).

 - `C-c C-f n`: Format item under cursor as note (`taskpaper-item-format-as-note`).


## Folding

The command `C-TAB` (`taskpaper-cycle`) changes the visibility of items in the buffer cycling through the most important states. The action of the command depends on the current cursor location.

When point is at the beginning of the buffer, rotate the entire buffer among the two states:

 - Overview: Only top-level items are shown.
 - Show All: Everything is shown.

When point is in an item, rotate current subtree among the three states:

 - Folded: Only the main item is shown.
 - Children: The item and its direct children are shown. From this state, you can move to one of the children and zoom in further.
 - Subtree: The entire subtree under the item is shown.

When Emacs first visits a TaskPaper file, the global state is set to Show All, i.e., all items are visible. This can be configured through the user option `taskpaper-startup-folded`.

The command `C-c *` (`taskpaper-outline-hide-other`) lets you focus on the current item under cursor. It hides everything except the current item with its ancestors and direct children. Other top-level items also shown to provide global context. The command `C-c C-a` (`taskpaper-outline-show-all`) unfolds all items at all levels (also bound to `ESC ESC`).


## Outline Navigation

The following commands jump to other items in the buffer.

 - `C-DOWN`: Go to the next item with the same level (`taskpaper-outline-forward-same-level`).

 - `C-UP`: Go to the previous item with the same level (`taskpaper-outline-backward-same-level`).

 - `S-UP`: Go to the parent item (`taskpaper-outline-up-level`).

 - `C-c C-j`: Go to selected item (`taskpaper-goto`).

The command `C-c C-j` (`taskpaper-goto`) prompts the user for an outline path to an item offering standard minibuffer completion for possible target locations. Special completion packages like [Ivy][emacs-ivy] or [Icicles][emacs-icicles] provide faster and more convenient way to select the outline path in minibuffer using regexp or fuzzy matching and incremental narrowing of possible selections. Additionally, you can use [Imenu][emacs-imenu] to go to a specific project in the buffer.


## Structure Editing

Structure editing commands let you easily rearrange the order and hierarchy of items in the outline.

Four main commands are provided for structure editing. The commands work on the current subtree (the current item plus all its children) and are assigned to the four arrow keys pressed with a modifier (META by default) in the following way.

 - `M-LEFT`: Promote the subtree under cursor by one level (`taskpaper-outline-promote-subtree`).

 - `M-RIGHT`: Demote the subtree under cursor by one level (`taskpaper-outline-demote-subtree`).

 - `M-UP`: Move the subtree under cursor up past the previous same-level subtree (`taskpaper-outline-move-subtree-up`).

 - `M-DOWN`: Move the subtree under cursor down past the next same-level subtree (`taskpaper-outline-move-subtree-down`).

The commands `M-LEFT` (`taskpaper-outline-promote-subtree`) and `M-RIGHT` (`taskpaper-outline-demote-subtree`) change the current subtree to a different outline level — i.e. the level of all items in the tree is decreased or increased. Note that the scope of "current subtree" may be changed after a promotion.

The commands `M-UP` (`taskpaper-outline-move-subtree-up`) and `M-DOWN` (`taskpaper-outline-move-subtree-down`) move the entire current subtree (folded or not) past the next same-level subtree in the given direction. The cursor moves with the subtree, so these commands can be used repeatedly to "drag" a subtree to the wanted position.

Additionally, the commands `TAB` (`taskpaper-outline-promote`) and `S-TAB` (`taskpaper-outline-demote`) promote/demote single item under cursor.

TaskPaper mode also provides following additional commands for working with subtrees (folded or not):

 - `C-c #`: Narrow buffer to the subtree under cursor (`taskpaper-narrow-to-subtree`).

 - `C-c C-m`: Mark the subtree under cursor (`taskpaper-mark-subtree`).

 - `C-c C-x v`: Copy all visible items in region to the kill ring and clipboard (`taskpaper-outline-copy-visible`).

 - `C-c C-x C-w`: Cut the subtree under cursor into the kill ring (`taskpaper-cut-subtree`).

 - `C-c C-x M-w`: Copy the subtree under cursor into the kill ring (`taskpaper-copy-subtree`).

 - `C-c C-x C-c`: Duplicate the subtree under cursor (`taskpaper-clone-subtree`).

 - `C-c C-x C-y`: Paste the subtree from the kill ring as child of the current item (`taskpaper-paste-subtree`).


## Tagging

In addition to the hierarchical ways of organizing your actions, you can also assign any number of tags to each task, project, or note. Tags provide another way to organize (and later search for) items. To create a tag type the `@` symbol preceded by a space and followed by a tag name with no spaces. Tag names may basically contain uppercase and lowercase letters, digits, hyphens, underscores, and dots. Tags can optionally have a value (or list of comma separated values) in parentheses after the tag name:

 - `@today`
 - `@priority(1)`
 - `@status(in press)`

The value text inside can have whitespaces, but no newlines. If you need to include parentheses in the tag value, precede them with a backslash.

After `@` symbol `M-TAB` offers in-buffer completion on tag names. The list of tags is created dynamically from all tags used in the current buffer. If your desktop intercepts the key binding `M-TAB` to switch windows, use `C-M-i` or `ESC TAB` as an alternative or customize your environment.

In addition to the in-buffer completion TaskPaper mode also implements another tag selection method called _fast tag selection_. This allows you to select your commonly used tags with just a single key press. For this to work you should assign unique, case-sensitive, letters to most of your commonly used tags. You can do this by configuring the user option `taskpaper-tag-alist` in your init file:

    (setq taskpaper-tag-alist
          '(("next"        . ?n)
            ("today"       . ?t)
            ("due(%^T)"    . ?d)
            ("added(%T)"   . ?a)
            ("priority(1)" . ?1)
            ("priority(2)" . ?2)
            ("priority(3)" . ?3)))

Pressing `C-c @` (`taskpaper-item-set-tag-fast-select`) will then present you with a special interface, listing all predefined tags with corresponding selection keys. Tag specifiers can have value in parentheses. The expression `%t` in the tag value is replaced with current date in [ISO 8601][iso8601-wiki] format, `%T` is replaced with current date and time, and `%^T` is like `%T`, but prompts the user for date & time.

If an item has a certain tag, all subitems will inherit the tag as well. To limit tag inheritance to specific tags, configure user option `taskpaper-tags-exclude-from-inheritance`.

The command `C-c C-r` (`taskpaper-remove-tag-at-point`) deletes single tag under cursor.


## Completing Tasks

Item is marked as completed by applying the `@done` tag. By default, items tagged with `@done` are visually crossed out.

The command `C-c C-d` (`taskpaper-item-toggle-done`) toggles done state for item under cursor. Alternatively you can toggle done state of the task by clicking on the task mark with `mouse-1` if the user option `taskpaper-pretty-marks` is non-nil. If the user option `taskpaper-complete-save-date` is non-nil, current date will be added to the `@done` tag. Additionally, you may specify a list of tags, which will be removed once the item is completed, using the user option `taskpaper-tags-to-remove-when-done`.


## Calendar Integration

The Emacs calendar created by Edward M. Reingold displays a three-month calendar with holidays from different countries and cultures. Following commands provide some integration with the calendar.

 - `C-c SPC`: Display current date or date under cursor in calendar (`taskpaper-show-in-calendar`).

 - `C-c >`: Access the calendar at the current date or date under cursor (`taskpaper-goto-calendar`).

 - `C-c <`: Insert a time stamp corresponding to the cursor date in the calendar (`taskpaper-date-from-calendar`).

 - `C-c .`: Prompt for a date and insert a corresponding time stamp (`taskpaper-read-date-insert-timestamp`).

The command `C-c >` (`taskpaper-goto-calendar`) goes to the calendar at the current date. If point is on a tag with value, interprets the value as date and goes to this date instead. With a `C-u` prefix, always goes to the current date.

The command `C-c .` (`taskpaper-read-date-insert-timestamp`) prompts for the date & time to insert at point. You can enter a date using date & time syntax described below. The current interpretation of your input will be displayed live in the minibuffer, right next to your input. If you find this distracting, turn the display off with the `taskpaper-read-date-display-live` user option.

Parallel to the minibuffer prompt, a calendar is popped up (see the user option `taskpaper-read-date-popup-calendar`). You can control the calendar from the minibuffer using the following commands:

 - `RET`: Choose date at cursor in calendar.

 - `mouse-1`: Select date by clicking on it.

 - ` >` / `<`: Scroll calendar forward/backward by one month.

 - `C-.`: Go to the current date.

 - `!`: Mark diary items in calendar.


## Date and Time Formats

These examples show the different formats that you can use when entering dates and times in the date & time prompt. The same formats can be used for date & time values in tags.

    - Do weekly review @due(Friday 12:30)
    - Attend meeting @due(2017-08-11 8 am)

If the time string is unparseable, current time is returned.


### Dates

Dates resolve to midnight of the given date.

 - `2017`
 - `2017-5`
 - `2017-05-10`
 - `--05-10`
 - `2017-W02`
 - `2017-W02-5`
 - `this week`
 - `next month`
 - `next Friday`
 - `last June 5`
 - `June`
 - `today`
 - `tomorrow`
 - `yesterday`

TaskPaper mode understands English month and weekday abbreviations. If you enter the name of a specific time period, the date will be at its beginning. So `June` without date means June first. Week refers to [ISO 8601][iso8601-wiki] standard week that starts on Monday, not Sunday.

You can refer to relative dates using common words (`today`, `tomorrow`, `yesterday`, `last week`, etc.). Words `this`, `next`, and `last` have specific meanings: `this Friday` always means the Friday in this week, `next Friday` always means the Friday in the next week, and `last Friday` always means the Friday in the last week, regardless of what day today is. Other units work in the same way.


### Times

Times are relative to the current date.

 - `6 am`
 - `6:45 pm`
 - `12:45`
 - `now`


### Duration Offsets

Duration offsets are relative to the current date.

 - `+2h`
 - `-1d`
 - `-4w`
 - `+2m`
 - `+1y`
 - `+2 hours`
 - `-1 day`
 - `-4 weeks`
 - `+2 months`
 - `+1 year`
 - `+3 Wed`

The shorthands `h`, `d`, `w`, `m`, and `y` stand for hour, day, week, month, and year, respectively. Positive numbers stand for the future whereas negative numbers stand for the past. Fractional (like `-0.5m`) and multiple (like `+2d +6h` or `+1y3m`) duration offsets are not supported.


### Combinations

You can combine dates, times, and duration offsets:

 - `tomorrow 8 am`
 - `last Jan 2 14:25 -1w`
 - `this month 8:00 +2 Fri`

Relative dates like `next Monday` should always be given as the _very first_ part of the time string. Duration offsets should always be given as the _very last_ part of the time string.


## Hyperlinks and Inline Images

TaskPaper mode auto-creates hyperlinks when it recognizes link text. Below are some examples of the links that will be recognized.

 - `https://www.taskpaper.com`
 - `mailto:username@domain.org`
 - `file:filename.txt`
 - `./filename.txt`
 - `file:///home/username/filename.txt`
 - `/home/username/filename.txt`
 - `file:/username@host:filename.txt`
 - `/username@host:filename.txt`

Absolute file links are starting with `/` or `~/`. Relative file links starting with `./` or `../` are relative to the location of your TaskPaper file. Spaces in file links must be protected using backslash, e.g., `./my\ file.txt`. File links to non-existing local files are highlighted using different face.

If the point is on a link the command `C-c C-o` or `mouse-1` (`taskpaper-open-link-at-point`) will launch a web browser for URLs (using `browse-url`) or start composing a mail message (using `compose-mail`). Furthermore, it will visit text and remote files in file links with Emacs and select a suitable application for local non-text files. Classification of files is based on file extension only. For non-specified extensions the system command to open files, like `open` on MS Windows and macOS, or the command specified in the mailcaps on GNU/Linux will be used. For more details see user options `taskpaper-file-apps` and `taskpaper-open-non-existing-files`.

The command `C-c C-l` (`taskpaper-insert-file-link-at-point`) inserts a file link at point offering standard minibuffer completion to select the name of the file. The path to the file is inserted relative to the directory of the current TaskPaper file, if the linked file is in the current directory or in a subdirectory of it, or if the path is written relative to the current directory using `../`. Otherwise an absolute path is used, if possible with `~/` for your home directory. You can force an absolute path with `C-u` prefix.

The command `C-c C-x C-v` (`taskpaper-toggle-inline-images`) toggles the inline display of linked images within the buffer skipping images larger than specified by `max-image-size`. Large images may be scaled down to fit in the buffer by setting `taskpaper-max-image-size` user option. Resizing works in Emacs v25 or higher built with ImageMagick support. You can ask for inline images to be displayed at startup by configuring the user option `taskpaper-startup-with-inline-images`.


## Sorting

The following commands sort same-level items. When point is at the beginning of the buffer, the top-level items are sorted. When point is in an item, the children of the current item are sorted. Sorting is case-insensitive. A `C-u` prefix will reverse the sort order.

 - `C-c C-s a`: Sort same-level items alphabetically (`taskpaper-sort-alpha`).

 - `C-c C-s t`: Sort same-level items by item type (`taskpaper-sort-by-type`).

When sorting is done, the hook `taskpaper-after-sorting-items-hook` is run. When children are sorted, hook functions are called with point on the parent item.

To meet more complex needs you can define your own sorting functions as described in [Scripting Guide][tp-mode-scripting-guide].


## Filtering

Filtering hides items that don't match the search creating a sparse tree, so that the entire document is folded as much as possible, but the selected information is made visible along with the outline hierarchy above it to provide minimal context.

The command `C-c /` (`taskpaper-occur`) prompts for a [regexp][emacs-regexp] and creates a sparse tree with all matches. Each match is also highlighted; the highlights disappear when the buffer is changed by an editing command, or by pressing `C-c C-c`.

The command `C-c C-t` (`taskpaper-search-tag-at-point`) will instantly show a filtered view of the items that contain the tag under cursor, shown in the context of higher level nodes. If the cursor is on the tag name, only the name is considered. If the cursor is on the tag value, both the name and value are considered. Alternatively, you can select the tag by clicking on it with `mouse-1`.

The command `C-c C-a` (`taskpaper-outline-show-all`) unfold all items at all levels (also bound to `ESC ESC`).


## Searching

You can create a sparse tree based on specific combinations of items' assigned tags and attributes. This is a great tool that lets you focus on the right things at the right time.

TaskPaper mode has a special mode for incremental querying. The I-query mode is entered by pressing `C-c C-i` (`taskpaper-iquery`). Query results are updated instantly as you type, creating a sparse tree with all matches. The command `C-c C-q` (`taskpaper-query`) is a non-incremental querying command, which requires you to type the entire query string before searching begins. This form of static, one-time querying (as opposed to incremental, on-the-fly querying) may be preferable in some situations, such as over slow network connections or on unusually large and deeply nested outlines, which may affect search responsiveness. You can limit your searches to a certain subtree or region by narrowing the buffer with `C-c #` (`taskpaper-narrow-to-subtree`) or `C-x n n` (`narrow-to-region`) respectively.

In addition to the standard motion and editing commands both static and incremental query modes define some additional key bindings while in minibuffer. Pressing `TAB` while editing query string offers completion on attribute names at point (see below). Pressing `C-c C-c` clears the query string and displays all items in the outline.

The query language syntax is described below.

__Note for TaskPaper app users:__ Though the query language syntax described here represents a valid subset of search syntax implemented in TaskPaper v3 app, the search behavior is slightly different. TaskPaper mode does not support item path syntax and set operations in search queries relying on tag inheritance instead. This behavior may change in future updates.


### Tags and Attributes

Every item in the outline has its own set of attributes. Explicit attributes are associated with assigned tags and have the same names, e.g., the tag `@priority(1)` added to an item will be translated to an attribute named "priority" whose value is "1". And vice versa, setting an explicit attribute programmatically will change the corresponding tag. TaskPaper mode also includes some implicit (built-in) read-only attributes, which are not associated with tags and set in other way:

 - `type`: Item's type (project, task, note, or blank)
 - `text`: Item's full line of text sans indentation


### Predicates

Search predicates describe what you are looking for. The full predicate pattern looks like this:

    @<attribute> <relation> [<modifier>] <search term>

Predicates start with the attribute, whose value or existence you want to test. Attribute represents tag with the same name or one of the implicit attributes and search term will be compared with the tag value. Pressing `TAB` offers completion on attribute names at point in both static and incremental query modes.

Relations determine the test that the predicate performs:

 - `=`: True if the attribute and search term are equal
 - `!=`: True if the attribute and search term are not equal
 - `<`: True if the attribute is less than the search term
 - `>`: True if the attribute is greater than the search term
 - `<=`: True if the attribute is less than or equal to the search term
 - `>=`: True if the attribute is greater than or equal to the search term
 - `contains`: True if the attribute contains the search term
 - `beginswith`: True if the attribute begins with the search term
 - `endswith`: True if the attribute ends with the search term
 - `matches`: True if the attribute matches the search term converted to a regular expression

The default type of comparison is case-insensitive string comparison. You can change this behavior by providing a modifier after the relation. The available modifiers are:

 - `i`: Case insensitive string compare (the default)
 - `s`: Case sensitive string compare
 - `n`: Numeric compare. Both sides of the compare are converted to numbers before comparing.
 - `d`: Date compare. Both sides of the compare are converted to dates before comparing.

Search terms can contain multiple words in sequence. Leading and trailing whitespaces are removed and multiple inter-word whitespaces collapsed into to a single space. If you want to search for an exact word or phrase preserving whitespaces, enclose the search term in double quotes. If some words or symbols are part of the predicate syntax or Boolean operators ("and", "or", "not", "matches", "@", etc.), they must be enclosed in double quotes. To include a double-quote character in a quoted search term, precede it with a backslash. If no search term is provided, attribute's presence will be tested.

Here are some examples for predicates:

 - `@today`
 - `@type = note`
 - `@due <=[d] +10d`
 - `@priority >[n] 3`
 - `@text endswith ?`
 - `@text matches "v[.0-9]"`
 - `@text contains[s] new logo`
 - `@text contains "this is not what I want"`
 - `@text contains "\"Winter\" by A. Vivaldi"`

You don't need to enter the entire predicate pattern every time you search. Predicates use default values when part of the pattern is missing. Attribute defaults to `text` and relation defaults to `contains`. For example the following predicates are equal:

 - `Inbox`
 - `@text Inbox`
 - `contains Inbox`
 - `@text contains Inbox`
 - `@text contains[i] Inbox`

__Note for TaskPaper app users:__ When using `matches` relation please keep in mind that TaskPaper v3 app searches use [JavaScript dialect][js-regexp] for regular expressions while TaskPaper mode accepts [Emacs dialect][emacs-regexp].


### Boolean Expressions

You can combine predicates with Boolean `and`, `or`, and `not` operators:

    @due <=[d] +14d and not @done or @today

Binary logical `and` binds more strongly than `or`. Unary logical `not` binds more strongly than both `and` and `or`. You can use parentheses to explicitly denote precedence by grouping parts of your query that should be evaluated first:

    @due <=[d] +14d and not (@done or @hold)


### Shortcuts

It's common to search TaskPaper items based on their type: project, task, or note.

For example you might want to find the "Inbox" project. The default way to do this would be:

    @type = project and Inbox

TaskPaper mode adds a shortcut for type based searches. The shortcut version of this search is:

    project Inbox

These are the shortcut forms and what they expand to:

 - `project` expands to `@type = project and`
 - `task` expands to `@type = task and`
 - `note` expands to `@type = note and`


### Storing Queries

Fast selection interface allows you to save your commonly used search queries and later select them with just a single key press. For this to work you should assign unique, case-sensitive, letters (or other characters, e.g., numbers) to your saved queries. You can do this by configuring the user option `taskpaper-custom-queries` in your init file:

    (setq taskpaper-custom-queries
          '((?w "Waiting"  "@waiting and not @done")
            (?d "Due Soon" "@due <=[d] +14d and not @done")))

The initial value in each item defines the key you have to press. The second parameter is a short description and the last one is the query string to be used for the matching. If the first element is a string, it will be used as block title to visually group queries. Pressing `C-c ?` (`taskpaper-query-fast-select`) will then present you with a special interface, listing all predefined queries with corresponding selection keys.


### Startup View

You can configure certain queries to be executed automatically when visiting a TaskPaper file. E.g., you can ask for all leaf notes (notes, which may content other notes, but no task or project items) to be folded at startup by adding following to your init file:

    (add-hook 'taskpaper-mode-hook
              '(lambda () (taskpaper-query "not @type = note")))


## Refiling

When reorganizing your outline, you may want to refile or to copy some of the items into a different subtree. Cutting, finding the right location, and then pasting the item can be a cumbersome task, especially for large outlines with many sublevels. The following special commands can be used to simplify this process:

 - `C-c C-w`: Move the subtree under cursor to different (possibly invisible) location (`taskpaper-refile-subtree`).

 - `C-c M-w`: Copy the subtree under cursor to different (possibly invisible) location (`taskpaper-refile-subtree-copy`).

The commands `C-c C-w` and `C-c M-w` offer possible target locations via outline path completion. This is the interface also used by the `C-c C-j` goto command.

The subtree is filed below the target item as a subitem. Depending on `taskpaper-reverse-note-order` setting, it will be either the first or last subitem.


## Archiving

When a project represented by a subtree is finished, you may want to move the tree out of the way.

The command `C-c C-x a` (`taskpaper-archive-subtree`) archives the subtree starting at the cursor position to the location given by `taskpaper-archive-location`. The default archive location is a file in the same directory as the current file, with the name derived by appending `_archive.taskpaper` to the current file name without extension. You can also choose what item to file archived items under. For details see the documentation string of the user option `taskpaper-archive-location`. The subtree is filed below the target item as a subitem. Depending on `taskpaper-reverse-note-order` setting, it will be either the first or last subitem. When the `taskpaper-archive-save-context` user option is non-nil, a `@project` tag with project hierarchy is added to the archived item.

When archiving the hook `taskpaper-archive-hook` runs after successfully archiving a subtree. Hook functions are called with point on the subtree in the original location. At this stage, the subtree has been added to the archive location, but not yet deleted from the original one.


## Multi-Document Support and Agenda View

For querying a collection of TaskPaper files, TaskPaper mode includes a powerful agenda mode. In this mode items from different TaskPaper files can be collected based on search queries and displayed in an organized way in a special agenda buffer. This buffer is read-only, but provides commands to visit the corresponding locations in the original TaskPaper files. In this way, all information is stored only once, removing the risk that your agenda view and agenda files may diverge.

The information to be shown is normally collected from all agenda files, the files listed in the user option `taskpaper-agenda-files`. If a directory is part of this list, all files with the extension `.taskpaper` in this directory will be part of the list. You can customize user option `taskpaper-agenda-file-regexp` to change this behavior. If `taskpaper-agenda-skip-unavailable-files` user option is non-nil, agenda mode will silently skip unavailable agenda files without issuing an error.

The following commands enter the agenda mode. The command `taskpaper-agenda-search` prompts the user for a search query. The command `taskpaper-agenda-select` let the user select a predefined query via the custom query dialog described above. You may consider to assign global key bindings to these commands in your init file:

    (global-set-key (kbd "C-c a") 'taskpaper-agenda-search)
    (global-set-key (kbd "C-c s") 'taskpaper-agenda-select)

Two user options control how the agenda buffer is displayed and whether the window configuration is restored when the agenda exits: `taskpaper-agenda-window-setup` and `taskpaper-agenda-restore-windows-after-quit`. For details see the documentation strings of these user options.


### Sorting Agenda Items

Before being inserted into an agenda buffer, the items are sorted. Sorting can be customized using the user option `taskpaper-agenda-sorting-predicate`. If the variable is `nil`, which is the default setting, agenda items just appear in the sequence in which they are found in the agenda files. The sorting predicate function is called with two arguments, the items to compare, and should return non-nil if the first item should sort before the second one.

In the example below items will be sorted according to their due dates. The sorting is done by date & time value (converted to float number of seconds since the beginning of the epoch). Items, which have no or empty `@due` tag, are assumed to have 2100-12-12 as due date, effectively ending up at the bottom of the sorted list.

    (setq taskpaper-agenda-sorting-predicate
          '(lambda (a b)
              (setq a (or (taskpaper-string-get-attribute a "due")
                          "2100-12-12")
                    b (or (taskpaper-string-get-attribute b "due")
                          "2100-12-12"))
              (taskpaper-time< a b)))

Items with equal sort keys maintain their relative order before and after the sort, which means, the `taskpaper-agenda-sorting-predicate` option can be accommodated to order items according to different criteria.


### Motion and Display Commands

Items in the agenda buffer are linked back to the TaskPaper file where they originate. You are not allowed to edit the agenda buffer itself, but commands are provided to show and jump to the original item location.

 - `n` or `DOWN`: Move to the next line (`taskpaper-agenda-next-line`).

 - `p` or `UP`: Move to the previous line (`taskpaper-agenda-previous-line`).

 - `SPC`: Display the original location of the item in another window (`taskpaper-agenda-show`).

 - `L`: Display the original location in another window and recenter that window (`taskpaper-agenda-show-recenter`).

 - `TAB`: Go to the original location of the item in another window (`taskpaper-agenda-goto`).

 - `RET`: Go to the original location of the item and delete other windows (`taskpaper-agenda-switch-to`).

 - `F`: Toggle Follow mode (`taskpaper-agenda-follow-mode`). In Follow mode, as you move the cursor through the agenda buffer, the other window always shows the corresponding location in the original TaskPaper file. The initial setting for this mode in new agenda buffers can be set with the user option `taskpaper-agenda-start-with-follow-mode`.

 - `c`: Display date under cursor in calendar (`taskpaper-show-in-calendar`).

 - `>`: Access calendar for the date under cursor (`taskpaper-goto-calendar`).

The command `SPC` (`taskpaper-agenda-show`) runs the hook `taskpaper-agenda-after-show-hook` after an item has been shown from the agenda. Hook functions are called with point in the buffer where the item originated. Thus, if you want to display only the current item, its ancestors and top-level items, put this in your init file:

    (setq taskpaper-agenda-after-show-hook
          'taskpaper-outline-hide-other)


### Filtering Agenda Items

You can also querying the agenda view to further narrow your search. Following commands and key bindings are defined in the agenda buffer:

 - `I`: Query agenda buffer using I-query mode (`taskpaper-iquery-mode`).

 - `Q`: Query agenda buffer non-interactively (`taskpaper-query`).

 - `S`: Query agenda buffer using custom query selection dialog (`taskpaper-query-fast-select`).

 - `t`: Show a filtered view of the items that contain the tag under cursor (`taskpaper-search-tag-at-point`).

 - `/`: Prompt for a regexp and show a filtered view with all matches highlighted (`taskpaper-occur`).

 - `C-c C-c`: Remove highlights (`taskpaper-occur-remove-highlights`).

 - `a`: Show all items (`taskpaper-outline-show-all`).


### Other Agenda Commands

 - `o`: Delete other windows (`delete-other-windows`).

 - `v`: Copy all visible items in region to the kill ring and clipboard (`taskpaper-outline-copy-visible`).

 - `r`: Recreate the agenda buffer (`taskpaper-agenda-redo`). Useful to reflect changes after modification of original TaskPaper files.

 - `q`: Quit agenda and remove the agenda buffer (`taskpaper-agenda-quit`).

 - `x`: Exit agenda and remove the agenda buffer and all buffers loaded by Emacs for the compilation of the agenda (`taskpaper-agenda-exit`). Buffers created by the user to visit TaskPaper files will not be removed.


## Miscellaneous

All the rest which did not fit elsewhere.

 - `M-x taskpaper-save-all-taskpaper-buffers RET`: Save all TaskPaper mode buffers without user confirmation.

 - `M-x taskpaper-mode-version RET`: Show TaskPaper mode version in the echo area.

 - `M-x taskpaper-mode-manual RET`: Browse TaskPaper mode user's manual.


# Customization

Although no configuration is necessary there are a few things that can be customized in addition to the options mentioned above. All configurations can be performed either via Emacs' Easy Customization Interface or by modifying Emacs' init files directly.


## Syntax Highlighting

You may specify special faces for specific tags using the user option `taskpaper-tag-faces`. For example:

    (setq taskpaper-tag-faces
          '(("start" . "green")
            ("today" . font-lock-warning-face)
            ("due"   . (:foreground "red" :weight bold))))

A string is interpreted as a color. The option `taskpaper-faces-easy-properties` determines if that color is interpreted as a foreground or a background color. For more details see the documentation string of the user option `taskpaper-tag-faces`.

Note: While using a list with face properties as shown for `due` _should_ work, this does not always seem to be the case. If necessary, define a special face and use that.

Additionally, the faces used for syntax highlighting can be modified to your liking by issuing `M-x customize-group RET taskpaper-faces RET`.

You can activate the task marks by setting the user option `taskpaper-pretty-marks` to non-nil, which makes the task marks appear as UTF-8 characters. This does not change the underlying buffer content, but it overlays the UTF-8 character _for display purposes only_. Tasks can then be marked as done by clicking on the task mark with `mouse-1`. The overlay characters for the task marks can be customized using the `taskpaper-bullet` and `taskpaper-bullet-done` user options.


## Cleaner Outline View

The `adaptive-wrap.el` sets the `wrap-prefix` correctly for indenting and wrapping of long-line items. The version included in this repository provides the `adaptive-wrap-prefix-mode` minor mode, which sets the `wrap-prefix` property on the fly so that single-long-line paragraphs get word-wrapped in a way similar to what you'd get with `M-q` using `adaptive-fill-mode`, but without actually changing the buffer's text. The present version supports tabs, works on one-line paragraphs, and can be used in modes other than TaskPaper mode. In can be activated globally by putting `adaptive-wrap-mode.el` on the load path and adding to your init file

    (require 'adaptive-wrap)
    (add-hook 'visual-line-mode-hook
              '(lambda () (adaptive-wrap-prefix-mode 1)))
    (global-visual-line-mode 1)


## Indentation Guides

If you want to display indentation guides in TaskPaper mode windows I recommend the [highlight-indent-guides.el][emacs-highlight-indent-guides] package. To enable it automatically when entering TaskPaper mode, you can use the `taskpaper-mode-hook`:

    (add-hook 'taskpaper-mode-hook
              '(lambda () (highlignt-indent-guides-mode 1)))

For customizing the way guides are displayed, see the package options.


# Acknowledgments

Thanks to Jesse Grosjean for writing [TaskPaper app][taskpaper] for macOS, whose functionality and sleekness I wanted to bring to Emacs, and for publishing TaskPaper's [open source model layer][birch-outline], which gave me some valuable implementation insights.

I would also thank the following people, from whose work TaskPaper mode has benefited greatly:

 - Carsten Dominik, Bastien Guerry and other Org mode developers for creating and maintaining [Org mode][emacs-orgmode] for Emacs, from which ideas and implementation I borrowed liberally;

 - Stephen Berman and Stefan Monnier for writing the original version of [adaptive-wrap.el][emacs-adaptive-wrap].


# Bugs

TaskPaper mode is developed and tested primarily for compatibility with GNU Emacs 24.3 and later. If you find any bugs in TaskPaper mode, please construct a test case or a patch and open a ticket on the [GitHub issue tracker][github-issues].


# License

This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 3, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <http://www.gnu.org/licenses/>.


[taskpaper]: https://www.taskpaper.com/

[taskpaper-guide]: https://guide.taskpaper.com/getting-started/

[tp-mode-scripting-guide]: ./scripting.md

[emacs-ivy]: https://github.com/abo-abo/swiper

[emacs-icicles]: https://www.emacswiki.org/emacs/Icicles

[emacs-imenu]: https://www.gnu.org/software/emacs/manual/html_node/emacs/Imenu.html

[iso8601-wiki]: https://en.wikipedia.org/wiki/ISO_8601

[emacs-regexp]: https://www.gnu.org/software/emacs/manual/html_node/elisp/Regular-Expressions.html

[js-regexp]: https://www.w3schools.com/jsref/jsref_obj_regexp.asp

[emacs-highlight-indent-guides]: https://github.com/DarthFennec/highlight-indent-guides

[emacs-orgmode]: http://orgmode.org/

[emacs-adaptive-wrap]: https://elpa.gnu.org/packages/adaptive-wrap.html

[birch-outline]: https://github.com/jessegrosjean/birch-outline

[github-issues]: https://github.com/saf-dmitry/taskpaper-mode/issues

