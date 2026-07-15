# 39 — Lovable Compatibility

> Lovable generates React + Vite + Tailwind + shadcn/ui + Framer Motion. Every pattern in this file is optimized for that exact stack.

---

## Lovable Stack

```
Framework:     React 18 + TypeScript
Build:         Vite
Styling:       Tailwind CSS v3
Components:    shadcn/ui (Radix UI primitives + Tailwind)
Icons:         lucide-react
Animations:    Framer Motion
State:         TanStack Query (server) + Zustand (client)
Routing:       React Router v6
Forms:         react-hook-form + Zod
```

---

## shadcn/ui Component Usage

```tsx
// Always import from @/components/ui — shadcn copies to your repo
import { Button }   from "@/components/ui/button";
import { Input }    from "@/components/ui/input";
import { Card, CardContent, CardHeader, CardTitle, CardDescription, CardFooter } from "@/components/ui/card";
import { Badge }    from "@/components/ui/badge";
import { Avatar, AvatarImage, AvatarFallback } from "@/components/ui/avatar";
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogDescription, DialogFooter, DialogTrigger } from "@/components/ui/dialog";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Switch }   from "@/components/ui/switch";
import { Slider }   from "@/components/ui/slider";
import { Separator } from "@/components/ui/separator";
import { Skeleton } from "@/components/ui/skeleton";
import { Toast, Toaster, useToast } from "@/components/ui/use-toast";
import { Sheet, SheetContent, SheetHeader, SheetTitle, SheetTrigger } from "@/components/ui/sheet";
import { Tooltip, TooltipContent, TooltipProvider, TooltipTrigger } from "@/components/ui/tooltip";
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger, DropdownMenuSeparator } from "@/components/ui/dropdown-menu";
import { Form, FormControl, FormDescription, FormField, FormItem, FormLabel, FormMessage } from "@/components/ui/form";
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table";
import { ScrollArea } from "@/components/ui/scroll-area";
import { Progress } from "@/components/ui/progress";
import { Command, CommandEmpty, CommandGroup, CommandInput, CommandItem, CommandList } from "@/components/ui/command";
import { Popover, PopoverContent, PopoverTrigger } from "@/components/ui/popover";
import { AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle, AlertDialogTrigger } from "@/components/ui/alert-dialog";
```

---

## cn() Utility

Always use `cn()` from `@/lib/utils` to merge Tailwind classes:

```tsx
import { cn } from "@/lib/utils";

// cn merges class names and handles Tailwind conflicts
<div className={cn(
  "rounded-lg border p-4",
  variant === "primary" && "bg-primary text-primary-foreground",
  variant === "secondary" && "bg-secondary text-secondary-foreground",
  className  // always accept and spread className prop
)} />
```

---

## shadcn/ui CSS Variables (Lovable Theme)

Lovable projects use HSL CSS custom properties. Always reference these:

```css
/* globals.css — Lovable's default token structure */
:root {
  --background:          0 0% 100%;
  --foreground:          222.2 84% 4.9%;
  --card:                0 0% 100%;
  --card-foreground:     222.2 84% 4.9%;
  --popover:             0 0% 100%;
  --popover-foreground:  222.2 84% 4.9%;
  --primary:             222.2 47.4% 11.2%;
  --primary-foreground:  210 40% 98%;
  --secondary:           210 40% 96.1%;
  --secondary-foreground:222.2 47.4% 11.2%;
  --muted:               210 40% 96.1%;
  --muted-foreground:    215.4 16.3% 46.9%;
  --accent:              210 40% 96.1%;
  --accent-foreground:   222.2 47.4% 11.2%;
  --destructive:         0 84.2% 60.2%;
  --destructive-foreground: 210 40% 98%;
  --border:              214.3 31.8% 91.4%;
  --input:               214.3 31.8% 91.4%;
  --ring:                222.2 84% 4.9%;
  --radius:              0.5rem;
}

.dark {
  --background:          222.2 84% 4.9%;
  --foreground:          210 40% 98%;
  /* ... */
}
```

```tsx
// Use Tailwind semantic classes, NOT arbitrary values:
// ❌ className="bg-[#2563eb] text-[#ffffff]"
// ✅ className="bg-primary text-primary-foreground"

// ❌ className="rounded-[6px] border-[1px]"
// ✅ className="rounded-md border"

// ❌ className="p-[16px]"
// ✅ className="p-4"
```

---

## Framer Motion Patterns

```tsx
import { motion, AnimatePresence, useMotionValue, useSpring, useTransform } from "framer-motion";

// Standard enter/exit animations
const fadeIn = {
  initial:  { opacity: 0, y: 8 },
  animate:  { opacity: 1, y: 0 },
  exit:     { opacity: 0, y: 8 },
  transition: { duration: 0.2, ease: [0, 0, 0.2, 1] },
};

// Page transition
const pageVariants = {
  initial:  { opacity: 0, x: -8 },
  animate:  { opacity: 1, x: 0 },
  exit:     { opacity: 0, x: 8 },
  transition: { duration: 0.2 },
};

// Stagger children
const containerVariants = {
  animate: { transition: { staggerChildren: 0.05 } },
};
const itemVariants = {
  initial: { opacity: 0, y: 16 },
  animate: { opacity: 1, y: 0 },
};

// Usage:
function AnimatedList({ items }) {
  return (
    <motion.ul variants={containerVariants} initial="initial" animate="animate">
      {items.map(item => (
        <motion.li key={item.id} variants={itemVariants}>
          {item.name}
        </motion.li>
      ))}
    </motion.ul>
  );
}

// AnimatePresence for mount/unmount
function Notification({ show, message }) {
  return (
    <AnimatePresence mode="wait">
      {show && (
        <motion.div
          key="notification"
          initial={{ opacity: 0, y: -16 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: -16 }}
          transition={{ duration: 0.15 }}
          className="notification"
        >
          {message}
        </motion.div>
      )}
    </AnimatePresence>
  );
}

// Spring physics for interactive elements
function SpringCard() {
  const x = useMotionValue(0);
  const y = useMotionValue(0);
  const rotateX = useTransform(y, [-100, 100], [15, -15]);
  const rotateY = useTransform(x, [-100, 100], [-15, 15]);
  
  return (
    <motion.div
      style={{ x, y, rotateX, rotateY, transformStyle: "preserve-3d" }}
      onMouseMove={e => {
        const rect = e.currentTarget.getBoundingClientRect();
        x.set(e.clientX - rect.left - rect.width  / 2);
        y.set(e.clientY - rect.top  - rect.height / 2);
      }}
      onMouseLeave={() => { x.set(0); y.set(0); }}
      className="card"
    />
  );
}

// Always respect prefers-reduced-motion
import { useReducedMotion } from "framer-motion";

function AccessibleAnimation({ children }) {
  const prefersReduced = useReducedMotion();
  
  return (
    <motion.div
      initial={prefersReduced ? false : { opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      transition={prefersReduced ? { duration: 0 } : { duration: 0.2 }}
    >
      {children}
    </motion.div>
  );
}
```

---

## Lucide Icons

```tsx
import {
  // Common UI
  X, Check, ChevronDown, ChevronUp, ChevronLeft, ChevronRight,
  Plus, Minus, MoreHorizontal, MoreVertical,
  Search, Filter, SortAsc, SortDesc,
  
  // Actions
  Edit, Trash2, Copy, Download, Upload, Share2, Send,
  Save, RefreshCw, RotateCcw,
  
  // Status
  AlertCircle, AlertTriangle, CheckCircle2, Info, XCircle,
  
  // Navigation
  Home, Settings, User, Users, Bell, LogOut,
  
  // Content
  File, Folder, Image, FileText, Link2, ExternalLink,
  
  // Communication
  Mail, MessageSquare, Phone,
  
  // Data
  BarChart3, TrendingUp, TrendingDown, PieChart,
  
  // Misc
  Star, Heart, Bookmark, Tag, Calendar, Clock, MapPin,
  Eye, EyeOff, Lock, Unlock, Shield, Globe, Zap,
} from "lucide-react";

// Icon sizing — always use className, not size prop for consistency
<ChevronRight className="h-4 w-4" />
<AlertCircle className="h-5 w-5 text-destructive" />
<CheckCircle2 className="h-6 w-6 text-green-500" />

// Accessibility
<button aria-label="Close dialog">
  <X className="h-4 w-4" aria-hidden="true" />
</button>
```

---

## TanStack Query (Data Fetching)

```tsx
import { useQuery, useMutation, useQueryClient, useInfiniteQuery } from "@tanstack/react-query";

// Fetch
function useProjects() {
  return useQuery({
    queryKey: ["projects"],
    queryFn:  () => api.get<Project[]>("/projects"),
    staleTime: 5 * 60 * 1000,  // 5 minutes
  });
}

// Mutation with optimistic update
function useDeleteProject() {
  const qc = useQueryClient();
  
  return useMutation({
    mutationFn: (id: string) => api.delete(`/projects/${id}`),
    onMutate: async (id) => {
      await qc.cancelQueries({ queryKey: ["projects"] });
      const prev = qc.getQueryData<Project[]>(["projects"]);
      qc.setQueryData<Project[]>(["projects"], old => old?.filter(p => p.id !== id));
      return { prev };
    },
    onError: (_, __, ctx) => {
      qc.setQueryData(["projects"], ctx?.prev);
      toast({ title: "Failed to delete project", variant: "destructive" });
    },
    onSettled: () => qc.invalidateQueries({ queryKey: ["projects"] }),
  });
}
```

---

## Lovable Toast Pattern

```tsx
import { useToast } from "@/components/ui/use-toast";

function ExampleComponent() {
  const { toast } = useToast();
  
  function onSuccess() {
    toast({
      title:       "Project created",
      description: "Your new project is ready.",
    });
  }
  
  function onError() {
    toast({
      title:       "Something went wrong",
      description: "Could not create project. Please try again.",
      variant:     "destructive",
    });
  }
  
  return (/* ... */);
}

// In app root — add <Toaster /> once
import { Toaster } from "@/components/ui/toaster";

function App() {
  return (
    <>
      <Router>...</Router>
      <Toaster />
    </>
  );
}
```

---

## Lovable File Structure

```
src/
├── components/
│   ├── ui/              ← shadcn/ui components (don't edit)
│   └── [feature]/       ← your feature components
├── hooks/               ← custom hooks
├── lib/
│   ├── utils.ts         ← cn() and helpers
│   └── api.ts           ← axios/fetch client
├── pages/               ← page components
├── stores/              ← Zustand stores
└── types/               ← TypeScript types
```

---

## Lovable Compatibility Checklist

- [ ] All UI components from shadcn/ui (`@/components/ui`)
- [ ] Icons from lucide-react only (no other icon libraries)
- [ ] `cn()` from `@/lib/utils` for class merging
- [ ] Tailwind semantic classes (`bg-primary`, not hex values)
- [ ] Animations via Framer Motion (`motion.div`, `AnimatePresence`)
- [ ] `useReducedMotion()` respected in all Framer Motion animations
- [ ] Data fetching via TanStack Query (`useQuery`, `useMutation`)
- [ ] Toasts via `useToast()` from shadcn
- [ ] `<Toaster />` placed once in app root
- [ ] Forms via react-hook-form + Zod + shadcn `<Form>`
- [ ] TypeScript strict mode — no `any`
- [ ] Dark mode via `.dark` class on `<html>`
