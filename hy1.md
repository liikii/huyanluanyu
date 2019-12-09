* How are classes laid out?
* How are data members accessed?
* How are member functions called?
* What is an adjuster thunk?
* What are the costs:
	* Of single, multiple, and virtual inheritance?
	* Of virtual functions and virtual function calls?
	* Of casts to bases, to virtual bases?
	* Of exception handling? 



http://hacksoflife.blogspot.com/2007/02/c-objects-part-1-basic-object-memory.html
C++ Objects Part 1: Basic Object Memory Layout
I spent a few minutes dissecting the C++ internal runtime structure for objects within Metrowerks CodeWarrior. There's some internal guts that C++ generates when you make objects; the format is not standardized between compilers, but looking at one implementation reveals some of the costs and implications of C++ objects and inheritance.

C++ breaks objects with methods down into data structures and functions, with function pointers used for virtual functions. Polymorphism depends on our ability to apply fixed unchanging code to diverse data structures and get meaningful execution. Therefore we can immediately postulate one rule that applies to all objects, no matter what C++ feature is used (inheritance, multiple inheritance, virtual base classes, etc): the in-memory layout of a pointer to class X must have a fixed layout no matter what real derived type the object has.

I'm only going to describe the case where objects have virtual functions and run-time type information. Based on the actual code and compiler settings, the compiler may sometimes eliminate these features, changing the in-memory structure, but that's a topic for another blog entry.

CodeWarrior (like virtually all C++ compilers, no pun intended) implements virtual methods via virtual function tables (vtables), which are static structures containing one function pointer for each virtual function. An object has a pointer to its class's vtable before any other object data; there is only one copy of the vtable per class - it's immutable and shared. (This makes objects smaller than having pointers to each virtual function in the actual object.)

The virtual function table actually contains two more entries before the function pointers: a pointer to a "type ID" structure, a separate structure that describes the class itself, and a memory offset whose use we'll look at later. So if we have this C++ code:

class foo {
virtual void a(int);
virtual void b(float);
int c;
float d;
};

The equivalent C code would look something like this:

struct foo_vtable {
typeid * type_id_ptr;
int offset;
void (* ptr_to_a)(int);
void (* ptr_to_b)(float);
};

void foo_a(int) { }
void foo_b(float) { }

static type_id foo_type_id = { ... };
static foo_vtable sfoo_vtable = { &foo_type_id, 0, foo_a, foo_b };

struct foo {
foo_vtable * vtable; // gets inited to &sfoo_vtable
int c;
float d;
};

Class type is stored separately - first ptr in the v-table. Motivation for this will be clear later.
Vtable has an offset - also understood later.