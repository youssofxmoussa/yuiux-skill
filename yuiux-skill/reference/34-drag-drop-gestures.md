# 34 — Drag, Drop & Gestures

> Drag and drop is powerful but fragile. It must feel direct and physical — items should stick to the cursor, drop zones must be obvious, and every drag operation needs a keyboard fallback.

---

## When to Use Drag and Drop

```
✅ Good candidates:
  - Reordering lists (task priority, column order)
  - Kanban boards (moving cards between columns)
  - File uploads (drag files onto drop zone)
  - Layout builders (arranging grid items)
  - Calendar events (reschedule by dragging)

❌ Poor candidates:
  - Primary navigation (clicking is faster and more accessible)
  - Selecting multiple items (use checkboxes)
  - Triggering actions that could be a button
  - Mobile-primary interfaces (swipe conflicts)
```

---

## @dnd-kit (Recommended)

@dnd-kit is the modern, accessible choice over react-beautiful-dnd (unmaintained) and HTML5 drag-and-drop (accessibility issues).

```bash
pnpm add @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities
```

### Sortable List

```tsx
import {
  DndContext, closestCenter, KeyboardSensor, PointerSensor,
  useSensor, useSensors, DragEndEvent, DragOverlay,
} from '@dnd-kit/core';
import {
  SortableContext, sortableKeyboardCoordinates,
  useSortable, verticalListSortingStrategy, arrayMove,
} from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';

// Sortable item component
function SortableItem({ id, label }: { id: string; label: string }) {
  const {
    attributes, listeners, setNodeRef,
    transform, transition, isDragging,
  } = useSortable({ id });
  
  return (
    <li
      ref={setNodeRef}
      style={{
        transform: CSS.Transform.toString(transform),
        transition,
        opacity: isDragging ? 0.5 : 1,
        zIndex:  isDragging ? 100 : undefined,
      }}
      className={`sortable-item ${isDragging ? 'dragging' : ''}`}
    >
      {/* Drag handle — separate from item content */}
      <button
        {...attributes}
        {...listeners}
        className="drag-handle"
        aria-label={`Drag to reorder ${label}`}
        type="button"
      >
        <GripVerticalIcon aria-hidden />
      </button>
      
      <span className="item-label">{label}</span>
      
      {/* Other actions: edit, delete, etc. */}
    </li>
  );
}

// Sortable list container
function SortableList({ initialItems }: { initialItems: Item[] }) {
  const [items, setItems] = useState(initialItems);
  const [activeId, setActiveId] = useState<string | null>(null);
  
  const sensors = useSensors(
    useSensor(PointerSensor, {
      activationConstraint: { distance: 8 }, // require 8px movement to start drag
    }),
    useSensor(KeyboardSensor, {
      coordinateGetter: sortableKeyboardCoordinates, // Space to pick up, arrows to move, Space/Enter to drop
    })
  );
  
  function handleDragStart(event: DragStartEvent) {
    setActiveId(event.active.id as string);
  }
  
  function handleDragEnd(event: DragEndEvent) {
    const { active, over } = event;
    
    if (over && active.id !== over.id) {
      setItems(items => {
        const oldIndex = items.findIndex(i => i.id === active.id);
        const newIndex = items.findIndex(i => i.id === over.id);
        const reordered = arrayMove(items, oldIndex, newIndex);
        
        // Persist new order
        saveOrder(reordered.map(i => i.id)).catch(() => {
          setItems(items); // rollback on failure
          toast.error('Failed to save order. Changes reverted.');
        });
        
        return reordered;
      });
    }
    
    setActiveId(null);
  }
  
  const activeItem = items.find(i => i.id === activeId);
  
  return (
    <DndContext
      sensors={sensors}
      collisionDetection={closestCenter}
      onDragStart={handleDragStart}
      onDragEnd={handleDragEnd}
      accessibility={{
        announcements: {
          onDragStart: ({ active }) => `Picked up ${getItemLabel(active.id)}. Use arrow keys to move, Space to drop.`,
          onDragOver: ({ active, over }) => over ? `Moving ${getItemLabel(active.id)} over ${getItemLabel(over.id)}.` : '',
          onDragEnd: ({ active, over }) => over ? `${getItemLabel(active.id)} dropped ${over ? 'successfully' : 'and returned to original position'}.` : '',
          onDragCancel: ({ active }) => `Dragging cancelled. ${getItemLabel(active.id)} returned to its original position.`,
        },
      }}
    >
      <SortableContext items={items.map(i => i.id)} strategy={verticalListSortingStrategy}>
        <ul className="sortable-list" aria-label="Sortable items. Use drag handles to reorder.">
          {items.map(item => (
            <SortableItem key={item.id} id={item.id} label={item.label} />
          ))}
        </ul>
      </SortableContext>
      
      {/* DragOverlay — renders dragged item above everything */}
      <DragOverlay>
        {activeItem && (
          <div className="sortable-item drag-overlay">
            <GripVerticalIcon aria-hidden />
            {activeItem.label}
          </div>
        )}
      </DragOverlay>
    </DndContext>
  );
}
```

---

## Kanban Board

```tsx
function KanbanBoard({ initialColumns }: { initialColumns: Column[] }) {
  const [columns, setColumns] = useState(initialColumns);
  const [activeCard, setActiveCard] = useState<Card | null>(null);
  
  const sensors = useSensors(
    useSensor(PointerSensor, { activationConstraint: { distance: 8 } }),
    useSensor(KeyboardSensor, { coordinateGetter: sortableKeyboardCoordinates })
  );
  
  const allCardIds = columns.flatMap(col => col.cards.map(c => c.id));
  const columnIds  = columns.map(col => col.id);
  
  function handleDragEnd(event: DragEndEvent) {
    const { active, over } = event;
    if (!over) { setActiveCard(null); return; }
    
    const activeColId = findColumnByCardId(active.id, columns)?.id;
    const overColId   = findColumnByCardId(over.id, columns)?.id ?? over.id;
    
    if (!activeColId) { setActiveCard(null); return; }
    
    setColumns(prev => {
      if (activeColId === overColId) {
        // Same column reorder
        return prev.map(col => {
          if (col.id !== activeColId) return col;
          const oldIdx = col.cards.findIndex(c => c.id === active.id);
          const newIdx = col.cards.findIndex(c => c.id === over.id);
          return { ...col, cards: arrayMove(col.cards, oldIdx, newIdx) };
        });
      }
      
      // Move to different column
      const card = prev.flatMap(c => c.cards).find(c => c.id === active.id)!;
      return prev.map(col => {
        if (col.id === activeColId) {
          return { ...col, cards: col.cards.filter(c => c.id !== active.id) };
        }
        if (col.id === overColId) {
          const overIdx = col.cards.findIndex(c => c.id === over.id);
          const insertAt = overIdx === -1 ? col.cards.length : overIdx;
          const newCards = [...col.cards];
          newCards.splice(insertAt, 0, card);
          return { ...col, cards: newCards };
        }
        return col;
      });
    });
    
    setActiveCard(null);
  }
  
  return (
    <DndContext sensors={sensors} onDragStart={e => setActiveCard(findCard(e.active.id, columns))} onDragEnd={handleDragEnd}>
      <div className="kanban-board" role="region" aria-label="Kanban board">
        <SortableContext items={columnIds} strategy={horizontalListSortingStrategy}>
          {columns.map(col => (
            <KanbanColumn key={col.id} column={col} allCardIds={allCardIds} />
          ))}
        </SortableContext>
      </div>
      
      <DragOverlay>
        {activeCard && <KanbanCardOverlay card={activeCard} />}
      </DragOverlay>
    </DndContext>
  );
}
```

---

## File Drop Zone

```tsx
function FileDropZone({
  onDrop,
  accept,
  maxSizeMb = 10,
  multiple = false,
}: FileDropZoneProps) {
  const [isDragging, setIsDragging] = useState(false);
  const [error, setError] = useState('');
  const inputRef = useRef<HTMLInputElement>(null);
  
  function validateFiles(files: FileList) {
    const validTypes = accept ? accept.split(',').map(t => t.trim()) : null;
    const maxBytes = maxSizeMb * 1024 * 1024;
    
    const valid: File[] = [];
    const errors: string[] = [];
    
    Array.from(files).forEach(file => {
      if (validTypes && !validTypes.some(t => {
        if (t.startsWith('.')) return file.name.endsWith(t);
        if (t.endsWith('/*')) return file.type.startsWith(t.slice(0, -2));
        return file.type === t;
      })) {
        errors.push(`${file.name}: Invalid file type`);
        return;
      }
      if (file.size > maxBytes) {
        errors.push(`${file.name}: Exceeds ${maxSizeMb}MB limit`);
        return;
      }
      valid.push(file);
    });
    
    if (errors.length) setError(errors.join(', '));
    else setError('');
    
    return valid;
  }
  
  function handleDrop(e: React.DragEvent) {
    e.preventDefault();
    setIsDragging(false);
    const valid = validateFiles(e.dataTransfer.files);
    if (valid.length) onDrop(valid);
  }
  
  return (
    <div
      className={`file-drop-zone ${isDragging ? 'dragging' : ''} ${error ? 'error' : ''}`}
      onDragEnter={e => { e.preventDefault(); setIsDragging(true); }}
      onDragLeave={e => { e.preventDefault(); setIsDragging(false); }}
      onDragOver={e => e.preventDefault()}
      onDrop={handleDrop}
      role="button"
      tabIndex={0}
      aria-label="File upload area. Drag and drop files here or press Enter to browse."
      onKeyDown={e => { if (e.key === 'Enter' || e.key === ' ') inputRef.current?.click(); }}
    >
      <UploadIcon aria-hidden />
      <p className="drop-zone-primary">
        {isDragging ? 'Drop files here' : 'Drag & drop files here'}
      </p>
      <p className="drop-zone-secondary">
        or{' '}
        <button
          type="button"
          className="drop-zone-browse"
          onClick={() => inputRef.current?.click()}
        >
          browse files
        </button>
        {accept && ` — ${accept} accepted`}
        {` — max ${maxSizeMb}MB`}
      </p>
      
      <input
        ref={inputRef}
        type="file"
        accept={accept}
        multiple={multiple}
        className="sr-only"
        onChange={e => {
          if (e.target.files?.length) {
            const valid = validateFiles(e.target.files);
            if (valid.length) onDrop(valid);
          }
        }}
        aria-hidden="true"
      />
      
      {error && <p role="alert" className="drop-zone-error">{error}</p>}
    </div>
  );
}
```

---

## Drag and Drop Accessibility

```
Keyboard interaction model for sortable lists:
  1. Tab to the drag handle button
  2. Space / Enter: Pick up the item
  3. Arrow Up/Down: Move the item
  4. Space / Enter: Drop the item at current position
  5. Escape: Cancel drag, return to original position

Screen reader announcements:
  - Pick up: "Picked up [item name]. Use arrow keys to move, Space to drop, Escape to cancel."
  - During move: "Moving [item name] over [target name]."
  - Drop:    "[Item name] dropped at position 3 of 10."
  - Cancel:  "Dragging cancelled. [Item name] returned to original position."
```

---

## Drag-and-Drop Checklist

- [ ] Drag handle separate from clickable item content
- [ ] 8px minimum drag distance before drag activates (prevents accidental drags)
- [ ] Keyboard sensor configured (Space/arrows/Escape workflow)
- [ ] `aria-label` on drag handle buttons ("Drag to reorder [item name]")
- [ ] Screen reader announcements for all drag lifecycle events
- [ ] DragOverlay shows visual "ghost" of dragged item
- [ ] Drop zones visually indicated while drag is active
- [ ] Optimistic reorder + rollback if server save fails
- [ ] Error toast if save fails after drop
- [ ] File drop zone has keyboard fallback (browse button)
- [ ] File drop zone validates type and size client-side
- [ ] File drop zone shows clear error for invalid files
- [ ] Mobile touch events handled (PointerSensor handles this)
