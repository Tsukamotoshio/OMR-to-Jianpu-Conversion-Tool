# File List Delete — Design Spec

**Date:** 2026-05-17
**Pages affected:** `gui/pages/jianpu_preview_page.py`, `gui/pages/score_preview_page.py`

---

## Goal

Add delete functionality to the file lists in the Jianpu preview page and Score preview page, consistent with the existing pattern in `gui/components/file_sidebar.py`.

---

## Interaction Design

### Per-row delete (single file)

- Add a `ft.IconButton(ft.Icons.CLOSE_ROUNDED)` as the last element in each `_make_item_row()` Container.
- Icon style matches `file_sidebar.py`: size 12, `ft.Colors.OUTLINE`, width/height 24.
- On click: show `ft.AlertDialog` with the file name and `[取消] [删除]` actions.
- On confirm: delete file(s) from disk, remove from `_pdf_paths` / `_mxl_paths`, rebuild list.

### Batch delete (checked files)

- Add a `_delete_checked_btn` (`ft.IconButton` with `ft.Icons.DELETE_OUTLINE_ROUNDED`) to the `sidebar_header` Row, placed between the export button and refresh button.
- Enabled only when `len(self._checked) > 0`; grayed out otherwise.
- On click: show `ft.AlertDialog` listing the count of files to delete.
- On confirm: delete all checked files, remove from internal lists, rebuild.

---

## Deletion Scope

### Jianpu preview page (`Output/*.pdf`)

Given stem `S` (where the PDF filename is `S_jianpu.pdf` or `S.pdf`):

| File | Directory | Condition |
|------|-----------|-----------|
| `S_jianpu.pdf` | `Output/` | always |
| `S.mid` | `Output/` | if exists |
| `S.jianpu.txt` | `editor-workspace/` | if exists |

The stem `S` is derived by stripping `_jianpu` suffix if present: already handled by the existing `display_name` logic — reuse `path.stem[:-len('_jianpu')]` where applicable.

### Score preview page (`xml-scores/*.mxl/.xml/.musicxml`)

| File | Directory | Condition |
|------|-----------|-----------|
| `<filename>` | `xml-scores/` | always |

No related files deleted. MXL/XML only.

---

## Post-delete Behavior

- If the deleted file(s) include `self._current_path`, clear the viewer and set `self._current_path = None`, then auto-select the next available file (if any).
- Update the select-all icon after any deletion.

---

## Dialog Text

| Scenario | Title | Content |
|----------|-------|---------|
| Single file (Jianpu) | `删除简谱文件` | `将永久删除：<name> 及其关联的 MIDI 和编辑文本。` |
| Single file (Score) | `删除五线谱文件` | `将永久删除：<name>` |
| Batch (Jianpu) | `批量删除简谱文件` | `将永久删除已勾选的 N 个简谱文件及其关联文件。` |
| Batch (Score) | `批量删除五线谱文件` | `将永久删除已勾选的 N 个五线谱文件。` |

Actions: `[取消]` (TextButton) · `[删除]` (FilledButton, error color)

---

## Implementation Notes

- Both pages already import `output_dir` and `editor_workspace_dir` (or `xml_scores_dir`). No new imports needed besides `Path` (already imported).
- `_delete_file_jianpu(path: Path)` — private helper on `JianpuPreviewPage`: deletes PDF + .mid + .jianpu.txt.
- `_delete_file_score(path: Path)` — private helper on `ScorePreviewPage`: deletes MXL/XML only.
- Both helpers use `path.unlink(missing_ok=True)` for atomic, safe deletion.
- Dialog pattern: identical to `file_sidebar.py` — `page.show_dialog()` / `page.pop_dialog()`.

---

## Files Changed

| File | Change |
|------|--------|
| `gui/pages/jianpu_preview_page.py` | Add × per-row + batch delete btn + `_delete_file_jianpu()` helper |
| `gui/pages/score_preview_page.py` | Add × per-row + batch delete btn + `_delete_file_score()` helper |
