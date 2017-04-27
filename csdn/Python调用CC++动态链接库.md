[cpp] view plain copy   //hello.h   #ifdef EXPORT_HELLO_DLL   #define
HELLO_API __declspec(dllexport)   #else   #define HELLO_API
__declspec(dllimport)   #endif      extern "C"   {       HELLO_API int
IntAdd(int , int);   }

