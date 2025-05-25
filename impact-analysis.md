# Problem Analysis

## Problem Context

The _Unsaved Changes_ `modal` dialog presents `button`s for
_Save_, _Don't Save_, _Cancel_, and these have `accelerator` shortcuts
`S`, `N` (and escape, though that's implicit).

## Problem Statement

The _Unsaved Changes_ accelerator shortcuts are not highlighted (underlined)
in the respective buttons of the modal dialog, but they should be.

## Software Problem Analysis

Accelerators are currently underlined in dropdown menus.
This is implemented in `tui.rs`:
- `menubar_menu_begin(text, accelerator)`
  - `menubar_label(text, accelerator)`
    - Looks for an exact or lowercase match
      ```rust
      // Add an underline to the accelerator.
      self.styled_label_add_text(&text[..off]);
      self.styled_label_set_attributes(Attributes::Underlined);
      self.styled_label_add_text(&text[off..off + 1]);
      self.styled_label_set_attributes(Attributes::None);
      self.styled_label_add_text(&text[off + 1..]);
      ```
    - Else:
      ```rust
      // Add the accelerator in parentheses and underline it.
      let ch = accelerator as u8;
      self.styled_label_add_text(text);
      self.styled_label_add_text("(");
      self.styled_label_set_attributes(Attributes::Underlined);
      self.styled_label_add_text(unsafe { str_from_raw_parts(&ch, 1) });
      self.styled_label_set_attributes(Attributes::None);
      self.styled_label_add_text(")");
      ```
The _Unsaved Changes_ dialog is implemented in `draw_editor.rs`:
- `draw_handle_wants_close(ctx, state)`
  - `ctx.modal_begin(...)`
  - Creates `ctx.button`s:
    ```rust
    if ctx.button("yes", loc(LocId::UnsavedChangesDialogYes)) {
        action = Action::Save;
    }
    ctx.inherit_focus();
    if ctx.button("no", loc(LocId::UnsavedChangesDialogNo)) {
        action = Action::Discard;
    }
    if ctx.button("cancel", loc(LocId::Cancel)) {
        action = Action::Cancel;
    }
    ```
    ... which do **not** support underlined accelerators.


# Solution Analysis

According to maintainer lhecker:
> The best approach for this would likely be to add a `shortcut_button` type
> to `tui.rs` and abstract away the existing logic for the menubar:
>
> [edit/src/tui.rs](https://github.com/microsoft/edit/blob/e8d40f6e7a95a6e19765ff19621cf0d39708f7b0/src/tui.rs#L3118-L3163)


