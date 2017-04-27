[cpp] view plain copy /** States:   stack_stop == NULL && stack_start == NULL:
did not start yet   stack_stop != NULL && stack_start == NULL:  already
finished   stack_stop != NULL && stack_start != NULL:  active  **/
//greenlet对象最终对应的数据的C结构体，这里可以理解为python对象的属性  typedef struct _greenlet {
PyObject_HEAD      char* stack_start;   //栈的顶部  将这里弄成null，标示已经结束了      char*
stack_stop;    //栈的底部      char* stack_copy;     //栈保存到的内存地址      intptr_t
stack_saved;   //栈保存在外面的大小      struct _greenlet* stack_prev;  //栈之间的上下层关系
struct _greenlet* parent;    //父对象      PyObject* run_info;   //其实也就是run对象
struct _frame* top_frame;   //这里可以理解为主要是控制python程序计数器      int
recursion_depth;   //栈深度      PyObject* weakreflist;      PyObject* exc_type;
PyObject* exc_value;      PyObject* exc_traceback;      PyObject* dict;  }
PyGreenlet;

