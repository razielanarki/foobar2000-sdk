Table of Contents

*   [foobar2000 v1.3 SDK readme](#foobar2000_v1.3_sdk_readme)

    *   [Compatibility](#compatibility)

    *   [Microsoft Visual Studio compatibility](#microsoft_visual_studio_compatibility)

    *   [Version 1.3 notes](#version_1.3_notes)

    *   [Basic usage](#basic_usage)

    *   [Structure of a component](#structure_of_a_component)

        *   [Services](#services)
        *   [Entrypoint services](#entrypoint_services)
        *   [Service extensions](#service_extensions)
        *   [Autopointer template use](#autopointer_template_use)
        *   [Exception use](#exception_use)
        *   [Storing configuration](#storing_configuration)
        *   [Use of global callback services](#use_of_global_callback_services)

    *   [Service class design guidelines (advanced)](#service_class_design_guidelines_advanced)

        *   [Cross-DLL safety](#cross-dll_safety)
        *   [Entrypoint service efficiency](#entrypoint_service_efficiency)

# [foobar2000 v1.3 SDK readme]()

<!-- SECTION "foobar2000 v1.3 SDK readme" [1-43] -->

## [Compatibility]()

Components built with this SDK are compatible with foobar2000 1.3. They are not compatible with any earlier versions (will fail to load), and not guaranteed to be compatible with any future versions, though upcoming releases will aim to maintain compatibility as far as possible without crippling newly added functionality.

<!-- SECTION "Compatibility" [44-395] -->

## [Microsoft Visual Studio compatibility]()

This SDK contains project files for Visual Studio 2010. This version of VS has been used for foobar2000 core development for a long time and it is the safest version to use.

VS2012, 2013 and 2015 currently produce incorrect output in release mode on foobar2000 code - see [forum thread](http://www.hydrogenaud.io/forums/index.php?showtopic=108411 "http://www.hydrogenaud.io/forums/index.php?showtopic=108411") for details. You can mitigate this issue by compiling all affected code (foobar2000 SDK code along with everything that uses foobar2000 SDK classes) with /d2notypeopt option.

<!-- SECTION "Microsoft Visual Studio compatibility" [396-972] -->

## [Version 1.3 notes]()

foobar2000 version 1.3 uses different metadb behavior semantics than older versions, to greatly improve the performance of multithreaded operation by reducing the time spent within global metadb locks.

Any methods that:

*   Lock the metadb - database\_lock() etc
*   Retrieve direct pointers to metadb info - get\_info\_locked() style

.. are now marked deprecated and implemented only for backwards compatibility; they should not be used in any new code.

It is recommended that you change your existing code using these to obtain track information using new get\_info\_ref() style methods for much better performance as these methods have minimal overhead and require no special care when used in multiple concurrent threads.

<!-- SECTION "Version 1.3 notes" [973-1728] -->

## [Basic usage]()

Each component must link against:

*   foobar2000\_SDK project (contains declarations of services and various service-specific helper code)
*   foobar2000\_component\_client project (contains DLL entrypoint)
*   shared.dll (various helper code, mainly win32 function wrappers taking UTF-8 strings)
*   PFC (non-OS-specific helper class library)

Optionally, components can use helper libraries with various non-critical code that is commonly reused across various foobar2000 components:

*   foobar2000\_SDK\_helpers - a library of various helper code commonly used by foobar2000 components.
*   foobar2000\_ATL\_helpers - another library of various helper code commonly used by foobar2000 components; requires WTL.

Foobar2000\_SDK, foobar2000\_component\_client and PFC are included in sourcecode form; you can link against them by adding them to your workspace and using dependencies. To link against shared.dll, you must add “shared.lib” to linker input manually.

Component code should include the following header files:

*   foobar2000.h from SDK - do not include other headers from the SDK directory directly, they're meant to be referenced by foobar2000.h only; it also includes PFC headers and shared.dll helper declaration headers.
*   Optionally: helpers.h from helpers directory (foobar2000\_SDK\_helpers project) - a library of various helper code commonly used by foobar2000 components.
*   Optionally: ATLHelpers.h from ATLHelpers directory (foobar2000\_ATL\_helpers project) - another library of various helper code commonly used by foobar2000 components; requires WTL. Note that ATLHelpers.h already includes SDK/foobar2000.h and helpers/helpers.h so you can replace your other include lines with a reference to ATLHelpers.h.

<!-- SECTION "Basic usage" [1729-3480] -->

## [Structure of a component]()

A component is a DLL that implements one or more entrypoint services and interacts with services provided by other components.

<!-- SECTION "Structure of a component" [3481-3647] -->

### [Services]()

A service type is an interface class, deriving directly or indirectly from `service_base` class. A service type class must not have any data members; it can only provide virtual methods (to be overridden by service implementation), helper non-virtual methods built around virtual methods, static helper methods, and constants / enums. Each service interface class must have a static `class_guid` member, used for identification when enumerating services or querying for supported functionality. A service type declaration should declare a class with public virtual/helper/static methods, and use `FB2K_MAKE_SERVICE_INTERFACE()` / `FB2K_MAKE_SERVICE_INTERFACE_ENTRYPOINT()` macro to implement standard service behaviors for the class; additionally, `class_guid` needs to be defined outside class declaration (e.g. `const GUID someclass::class_guid = {….};` ). Note that most of components will not declare their own service types, they will only implement existing ones declared in the SDK.

A service implementation is a class derived from relevant service type class, implementing virtual methods declared by service type class. Note that service implementation class does not implement virtual methods declared by service\_base; those are implemented by service type declaration framework (`service_query`) or by instantiation framework (`service_add_ref` / `service_release`). Service implementation classes are instantiated using `service_factory` templates in case of entrypoint services (see below), or using `service_impl_t` template and operator `new`: ”`myserviceptr = new service_impl_t<myservice_impl>(params);`”.

Each service object provides reference counter features and (`service_add_ref()` and `service_release()` methods) as well as a method to query for extended functionality (`service_query()` method). Those methods are implemented by service framework and should be never overridden by service implementations. These methods should also never be called directly - reference counter methods are managed by relevant autopointer templates, `service_query_t` function template should be used instead of calling `service_query` directly, to ensure type safety and correct type conversions.

<!-- SECTION "Services" [3648-5913] -->

### [Entrypoint services]()

An entrypoint service type is a special case of a service type that can be registered using `service_factory` templates, and then accessed from any point of service system (excluding DLL startup/shutdown code, such as code inside static object constructors/destructors). An entrypoint service type class must directly derive from `service_base`.

Registered entrypoint services can be accessed using:

*   For services types with variable number of implementations registered: `service_enum_t<T>` template, `service_class_helper_t<T>` template, etc, e.g.

        service_enum_t<someclass> e; service_ptr_t<someclass> ptr; while(e.next(ptr)) ptr->dosomething();

*   For services types with single always-present implementation registered - such as core services like `playlist_manager` - using `static_api_ptr_t<T>` template, e.g.:

        static_api_ptr_t<someclass> api; api->dosomething(); api->dosomethingelse();

*   Using per-service-type defined static helper functions, e.g. `someclass::g_dosomething()` - those use relevant service enumeration methods internally.

An entrypoint service type must use `FB2K_MAKE_SERVICE_INTERFACE_ENTRYPOINT()` macro to implement standard entrypoint service behaviors, as opposed to all other service types that use `FB2K_MAKE_SERVICE_INTERFACE()` macro instead.

You can register your own entrypoint service implementations using either `service_factory_t` or `service_factory_single_t` template - the difference between the two is that the former instantiates the class on demand, while the latter keeps a single static instance and returns references to it when requested; the latter is faster but usable only for things that have no per-instance member data to maintain. Each `service_factory_t` / `service_factory_single_t` instance should be a static variable, such as: ”`static service_factory_t<myclass> g_myclass_factory;`”.

Certain service types require custom `service_factory` helper templates to be used instead of standard `service_factory_t` / `service_factory_single_t` templates; see documentation of specific service type for exact info about registering.

A typical entrypoint service implementation looks like this:

    class myservice_impl : public myservice {
    public:
    	void dosomething() {....};
    	void dosomethingelse(int meh) {...};
    };
    static service_factory_single_t<myservice_impl> g_myservice_impl_factory;

<!-- SECTION "Entrypoint services" [5914-8385] -->

### [Service extensions]()

Additional methods can be added to any service type, by declaring a new service type class deriving from service type class you want to extend. For example:

    class myservice : public service_base { public: virtual void dosomething() = 0; FB2K_MAKE_SERVICE_INTERFACE_ENTRYPOINT(myservice); };
    class myservice_v2 : public myservice { public: virtual void dosomethingelse() = 0; FB2K_MAKE_SERVICE_INTERFACE(myservice_v2, myservice); };

In such scenario, to query whether a myservice instance is a `myservice_v2` and to retrieve `myservice_v2` pointer, use `service_query` functionality:

    service_ptr_t<myservice> ptr;
    (...)
    service_ptr_t<myservice_v2> ptr_ex;
    if (ptr->service_query_t(ptr_ex)) { /* ptr_ex is a valid pointer to myservice_v2 interface of our myservice instance */ (...) }
    else {/* this myservice implementation does not implement myservice_v2 */ (...) }

<!-- SECTION "Service extensions" [8386-9325] -->

### [Autopointer template use]()

When performing most kinds of service operations, `service_ptr_t<T>` template should be used rather than working with service pointers directly; it automatically manages reference counter calls, ensuring that the service object is deleted when it is no longer referenced. Additionally, `static_api_ptr_t<T>` can be used to automatically acquire/release a pointer to single-implementation entrypoint service, such as one of standard APIs like `playlist_manager`.

<!-- SECTION "Autopointer template use" [9326-9830] -->

### [Exception use]()

Most of API functions use C++ exceptions to signal failure conditions. All used exception classes must derive from `std::exception` (which `pfc::exception` is typedef'd to); this design allows various instances of code to use single `catch()` line to get human-readable description of the problem to display.

Additionally, special subclasses of exceptions are defined for use in specific conditions, such as `exception_io` for I/O failures. As a result, you must provide an exception handler whenever you invoke any kind of I/O code that may fail, unless in specific case calling context already handles exceptions (e.g. input implementation code - any exceptions should be forwarded to calling context, since exceptions are a part of input API).

Implementations of global callback services such as `playlist_callback`, `playback_callback` or `library_callback` must not throw unhandled exceptions; behaviors in case they do are undefined (app termination is to be expected).

<!-- SECTION "Exception use" [9831-10848] -->

### [Storing configuration]()

In order to create your entries in the configuration file, you must instantiate some objects that derive from `cfg_var` class. Those can be either predefined classes (`cfg_int`, `cfg_string`, etc) or your own classes implementing relevant methods.

Each `cfg_var` instance has a GUID assigned, to identify its configuration file entry. The GUID is passed to its constructor (which implementations must take care of, typically by providing a constructor that takes a GUID and forwards it to `cfg_var` constructor).

Note that `cfg_var` objects can only be instantiated statically (either directly as static objects, or as members of other static objects). Additionally, you can create configuration data objects that can be accessed by other components, by implementing `config_object` service. Some standard configuration variables can be also accessed using `config_object` interface.

<!-- SECTION "Storing configuration" [10849-11784] -->

### [Use of global callback services]()

Multiple service classes presented by the SDK allow your component to receive notifications about various events:

*   file\_operation\_callback - tracking file move/copy/delete operations.
*   library\_callback - tracking Media Library content changes.
*   metadb\_io\_callback - tracking tag read / write operations altering cached/displayed media information.
*   play\_callback - tracking playback related events.
*   playback\_statistics\_collector - collecting information about played tracks.
*   playlist\_callback, playlist\_callback\_single - tracking playlist changes (the latter tracks only active playlist changes).
*   playback\_queue\_callback - tracking playback queue changes.
*   titleformat\_config\_callback - tracking changes of title formatting configuration.
*   ui\_drop\_item\_callback - filtering items dropped into the UI.

All of global callbacks operate only within main app thread, allowing easy cooperation with windows GUI - for an example, you perform playlist view window repainting directly from your playlist\_callback implementation.

#### [Global callback recursion issues]()

There are restrictions on things that are legal to call from within global callbacks. For an example, you can't modify a playlist from inside a playlist callback, because there are other registered callbacks tracking playlist changes that haven't been notified about the change being currently processed yet.

You must not enter modal message loops from inside global callbacks, as those allow any unrelated code (queued messages, user input, etc.) to be executed, without being aware that a global callback is being processed. Certain global API methods such as metadb\_io::load\_info\_multi or threaded\_process::run\_modal enter modal loops when called. Use main\_thread\_callback service to avoid this problem and delay execution of problematic code.

You should also avoid firing a cross-thread SendMessage() inside global callbacks as well as performing any operations that dispatch global callbacks when handling a message that was sent through a cross-thread SendMessage(). Doing so may result in rare unwanted recursions - SendMessage() call will block the calling thread and immediately process any incoming cross-thread SendMessage() messages. If you're handling a cross-thread SendMessage() and need to perform such operation, delay it using PostMessage() or main\_thread\_callback.

<!-- SECTION "Use of global callback services" [11785-14206] -->

## [Service class design guidelines (advanced)]()

This chapter describes things you should keep on your mind when designing your own service type classes. Since 99% of components will only implement existing service types rather than adding their own cross-DLL-communication protocols, you can probably skip reading this chapter.

<!-- SECTION "Service class design guidelines (advanced)" [14207-14543] -->

### [Cross-DLL safety]()

It is important that all function parameters used by virtual methods of services are cross-DLL safe (do not depend on compiler-specific or runtime-specific behaviors, so no unexpected behaviors occur when calling code is built with different compiler/runtime than callee). To achieve this, any classes passed around must be either simple objects with no structure that could possibly vary with different compilers/runtimes (i.e. make sure that any memory blocks are freed on the side that allocated them); easiest way to achieve this is to reduce all complex data objects or classes passed around to interfaces with virtual methods, with implementation details hidden from callee. For an example, use `pfc::string_base&` as parameter to a function that is meant to return variable-length strings.

<!-- SECTION "Cross-DLL safety" [14544-15371] -->

### [Entrypoint service efficiency]()

When designing an entrypoint service interface meant to have multiple different implementations, you should consider making it possible for all its implementations to use `service_factory_single_t` (i.e. no per-instance member data); by e.g. moving functionality that needs multi-instance operation to a separate service type class that is created on-demand by one of entrypoint service methods. For example:

    class myservice : public service_base {
    public:
    	//this method accesses per-instance member data of the implementation class
    	virtual void workerfunction(const void * p_databuffer,t_size p_buffersize) = 0;
    	//this method is used to determine which implementation can be used to process specific data stream.
    	virtual bool queryfunction(const char * p_dataformat) = 0;
     
    	FB2K_MAKE_SERVICE_INTERFACE_ENTRYPOINT(myservice);
    };
    (...)
    service_ptr_t<myservice> findservice(const char * p_dataformat) {
    	service_enum_t<myservice> e; service_ptr_t<myservice> ptr;
    	//BOTTLENECK, this dynamically instantiates the service for each query.
    	while(e.next(ptr)) {
    		if (ptr->queryfunction(p_dataformat)) return ptr;
    	}
    	throw exception_io_data();
    }

.. should be changed to:

    //no longer an entrypoint service - use myservice::instantiate to get an instance instead.
    class myservice_instance : public service_base {
    public:
    	virtual void workerfunction(const void * p_databuffer,t_size p_buffersize) = 0;
    	FB2K_MAKE_SERVICE_INTERFACE(myservice_instance,service_base);
    };
     
    class myservice : public service_base {
    public:
    	//this method is used to determine which implementation can be used to process specific data stream.
    	virtual bool queryfunction(const char * p_dataformat) = 0;
    	virtual service_ptr_t<myservice_instance> instantiate() = 0;
    	FB2K_MAKE_SERVICE_INTERFACE_ENTRYPOINT(myservice);
    };
     
    template<typename t_myservice_instance_impl>
    class myservice_impl_t : public myservice {
    public:
    	//implementation of myservice_instance must provide static bool g_queryformatfunction(const char*);
    	bool queryfunction(const char * p_dataformat) {return t_myservice_instance_impl::g_queryfunction(p_dataformat);}
    	service_ptr_t<myservice_instance> instantiate() {return new service_impl_t<t_myservice_instance_impl>();}
    };
     
    template<typename t_myservice_instance_impl> class myservice_factory_t :
    	public service_factory_single_t<myservice_impl_t<t_myservice_instance_impl> > {};
    //usage: static myservice_factory_t<myclass> g_myclass_factory;
     
    (...)
     
    service_ptr_t<myservice_instance> findservice(const char * p_dataformat) {
    	service_enum_t<myservice> e; service_ptr_t<myservice> ptr;
    	//no more bottleneck, enumerated service does not perform inefficient operations when requesting an instance.
    	while(e.next(ptr)) {
    		//"inefficient" part is used only once, with implementation that actually supports what we request.
    		if (ptr->queryfunction(p_dataformat)) return ptr->instantiate();
    	}
    	throw exception_io_data();
    }

<!-- SECTION "Entrypoint service efficiency" [15372-] -->
