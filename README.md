# Inventory-managemnt-by-sakshi
It's a simple inventory management system made in c++
#Inventory Manager

A console application that stores inventory items in a binary file (`inventory.dat`) and exposes full CRUD operations through an interactive menu.

- **C layer** (`src/inventory.c`, `include/inventory.h`) — handles all binary file I/O using `fread` / `fwrite` / `fseek`.
- **C++ layer** (`include/InventoryManager.hpp`, `src/main.cpp`) — wraps the C API in a class, uses `std::vector` + `std::sort`, and drives the interactive menu.

---

## File Structure

```
inventory_manager/
├── include/
│   ├── inventory.h          # C struct + function declarations (extern "C")
│   └── InventoryManager.hpp # C++ class wrapping the C API
├── src/
│   ├── inventory.c          # C backend — binary file storage
│   └── main.cpp             # C++ menu, input validation, entry point
├── Makefile
├── CMakeLists.txt
└── README.md
```

---

## Build & Run

### Option A — Make (recommended)

```bash
cd inventory_manager
make          # compiles into ./inventory_manager
./inventory_manager
```

Clean everything (including the data file):

```bash
make clean
```

### Option B — CMake

```bash
cd inventory_manager
mkdir build_cmake && cd build_cmake
cmake ..
make
./inventory_manager
```

### Requirements

| Tool | Minimum version |
|------|----------------|
| gcc  | 5.0 (C11)      |
| g++  | 7.0 (C++17)    |
| make | 3.81           |

---

## Usage

After launching, a numbered menu is displayed:

```
==============================
     INVENTORY MANAGER
==============================
  1  Add item
  2  View item by ID
  3  Update item
  4  Delete item
  5  List all items
  6  Exit
------------------------------
```

All input is validated — invalid values produce a message and re-prompt (no crash).

---

## Test Cases

1. **Persistence across restarts**
   - Add items with IDs 1, 2, 3 (names: "Hammer", "Wrench", "Screwdriver").
   - Select **6 Exit**, restart the program.
   - Select **5 List all** → all three items appear.
   - ✅ Data survives process exit.

2. **Duplicate ID rejection**
   - With items already in the file, attempt **1 Add item** with an ID that already exists.
   - Expected: `[!] Failed — ID X may already exist.`
   - ✅ Duplicate is refused; existing record is unchanged.

3. **Update persists after restart**
   - **3 Update item** on ID 2 → change name to "Monkey Wrench", quantity to 50, price to 19.99.
   - Exit and restart.
   - **2 View item** for ID 2 → displays updated values.
   - ✅ Update is durable.

4. **Soft delete**
   - **4 Delete item** on ID 3.
   - **5 List all** → ID 3 does not appear.
   - **2 View item** for ID 3 → `[!] Item 3 not found (or deleted).`
   - Exit and restart; repeat list/view → still absent.
   - ✅ Deleted items are invisible in all views.

5. **Input validation**
   - At any prompt, enter `-5` for ID → re-prompted.
   - Enter a blank name → re-prompted.
   - Enter `-1` for quantity → re-prompted.
   - Enter `-9.99` for price → re-prompted.
   - ✅ Invalid input is caught; the program never crashes.

---

## Design Notes

- The binary file stores raw `Item` structs sequentially. Each record is exactly `sizeof(Item)` bytes, so `fseek(fp, n * sizeof(Item), SEEK_SET)` navigates directly to any record — O(1) seek for update/delete.
- Soft delete (`is_deleted = 1`) keeps the file layout stable; IDs are never recycled.
- `list_items` buffers up to 1 024 active records into a `std::vector`, which is then `std::sort`-ed before printing.
- 
