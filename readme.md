Table of Contents

*   [foobar2000 v1.6 SDK readme](#foobar2000_v1.6_sdk_readme)

    *   [Compatibility](#compatibility)

    *   [Microsoft Visual Studio compatibility](#microsoft_visual_studio_compatibility)

        *   [Ill behavior of Visual C whole program optimization](#ill_behavior_of_visual_c_whole_program_optimization)
        *   ["virtual memory range for PCH exceeded" error](#virtual_memory_range_for_pch_exceeded_error)

    *   [Version 1.6 notes](#version_1.6_notes)

        *   [Windows XP support](#windows_xp_support)
        *   [Audio output API extensions](#audio_output_api_extensions)
        *   [WebP support, imageLoaderLite](#webp_support_imageloaderlite)
        *   [File cache utilities](#file_cache_utilities)
        *   [Cuesheet wrapper fixes](#cuesheet_wrapper_fixes)
        *   [libPPUI](#libppui)

    *   [Version 1.5 notes](#version_1.5_notes)

        *   [libPPUI](#libppui1)

    *   [Version 1.4 notes](#version_1.4_notes)

        *   [Namespace cleanup](#namespace_cleanup)
        *   [Decoders](#decoders)
        *   [Dynamic runtime](#dynamic_runtime)
        *   [service\_query()](#service_query)

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

# [foobar2000 v1.6 SDK readme]()

<!-- SECTION "foobar2000 v1.6 SDK readme" [1-42] -->

## [Compatibility]()

Components built with this SDK are compatible with foobar2000 1.4 and newer. They are not compatible with any earlier versions (will fail to load), and not guaranteed to be compatible with any future versions, though upcoming releases will aim to maintain compatibility as far as possible without crippling newly added functionality.

You can alter the targeted foobar2000 API level, edit SDK/foobar2000.h and change the value of FOOBAR2000\_TARGET\_VERSION.

Currently supported values are 79 (for 1.4 series) and 80 (for 1.5 & 1.6 series).

<!-- SECTION "Compatibility" [43-610] -->

## [Microsoft Visual Studio compatibility]()

This SDK contains project files for Visual Studio 2017.

<!-- SECTION "Microsoft Visual Studio compatibility" [611-718] -->

### [Ill behavior of Visual C whole program optimization]()

Visual Studio versions from 2012 up produce incorrect output with default release settings on foobar2000 code - see [forum thread](http://www.hydrogenaud.io/forums/index.php?showtopic=108411 "http://www.hydrogenaud.io/forums/index.php?showtopic=108411") for details. You can mitigate this issue by compiling all affected code (foobar2000 SDK code along with everything that uses foobar2000 SDK classes) with /d2notypeopt option. This is set by default on project files provided with the SDK; please make sure that you set this option on any projects of your own.

If you're aware of a better workaround - such as a source code change rather than setting an obscure compiler flag - please let us know; posting on the forum is preferred for the benefit of other users of this SDK.

<!-- SECTION "Ill behavior of Visual C whole program optimization" [719-1498] -->

### ["virtual memory range for PCH exceeded" error]()

By convention, foobar2000 code used to #include every single SDK and utility header in a precompiled header (PCH) file. This became an issue with certain Visual Studio setups, as the PCH payload became extremely large.

The SDK has been changed to reduce the amount of unnecessary code shoved into #included headers; errors of this type are yet to be seen with this version of the foobar2000 SDK compiled under VS2017.

If you run into this error, we recommend trimming down the #includes and only referencing specific headers from helpers instead of #including all of them (via old helpers.h).

<!-- SECTION "virtual memory range for PCH exceeded error" [1499-2150] -->

## [Version 1.6 notes]()

<!-- SECTION "Version 1.6 notes" [2151-2180] -->

### [Windows XP support]()

The SDK project files are configured for targetting Windows 7 and newer, with SSE2 level instruction set.

If you really must compile components that run on 20 years old computers, you can still change relevant settings, or use an older SDK. Please test your components on actual hardware if you do so, VS2017 compiler has been known to output SSE opcodes when specifically told not to.

<!-- SECTION "Windows XP support" [2181-2597] -->

### [Audio output API extensions]()

New methods have been added to the output API, such as reporting whether your output is low-latency or high-latency, to interop with the new fading code.

foobar2000 now provides a suite of special fixes for problematic output implementations (guarenteed update() calls in regular intervals, silence injected at the end); use flag\_needs\_shims to enable with your output. While such feature might seem unnecessary, it allowed individual outputs to be greatly simplified, removing a lot of other special fixes implemented per output.

<!-- SECTION "Audio output API extensions" [2598-3169] -->

### [WebP support, imageLoaderLite]()

Use fb2k::imageLoaderLite API to load a Gdiplus image from a memory block, or to extract info without actually decoding the picture. It supports WebP payload and will be updated with any formats added in the future.

<!-- SECTION "WebP support, imageLoaderLite" [3170-3426] -->

### [File cache utilities]()

New interface has been introduced - read\_ahead\_tools - to access advanced playback settings for read-ahead cache; primarily needed if your decoder needs to open additional files over the internet.

<!-- SECTION "File cache utilities" [3427-3655] -->

### [Cuesheet wrapper fixes]()

Wrapper code to support embedded cuesheets on arbitrary formats has been updated to deal with remote files gracefully, that is not present any chapters on such.

<!-- SECTION "Cuesheet wrapper fixes" [3656-3850] -->

### [libPPUI]()

Notable changes: \* WebP vs Gdiplus interop (in case you want to load WebP into Gdiplus in your own app - in fb2k, use imageLoaderLite) \* Fixed horrible GdiplusImageFromMem() bug \* Combo boxes in CListControl \* CListControl graying/disabling of individual items

<!-- SECTION "libPPUI" [3851-4130] -->

## [Version 1.5 notes]()

<!-- SECTION "Version 1.5 notes" [4131-4160] -->

### [libPPUI]()

Various code from the helpers projects that was in no way foobar2000 specific became libPPUI. In addition, Default User Interface list control has been thrown in. libPPUI is released under a non-restrictive license. Reuse in other projects - including commercial projects - is encouraged. Credits in binary redistribution are not required.

Existing foobar2000 components that reference SDK helpers/ATLHelpers will need updating to reference libPPUI instead. Separate helpers/ATLHelpers projects are no more, as all projects depend on libPPUI which requires ATL/WTL.

<!-- SECTION "libPPUI" [4161-4746] -->

## [Version 1.4 notes]()

<!-- SECTION "Version 1.4 notes" [4747-4776] -->

### [Namespace cleanup]()

Some very old inconsistencies in the code have been cleaned up. Various bit\_array classes are now in pfc namespace where they belong. Please use pfc::bit\_array and so on in new code. If you have code that references bit\_array classes without the pfc:: prefix, put “using pfc::bit\_array” in your headers to work around it.

<!-- SECTION "Namespace cleanup" [4777-5126] -->

### [Decoders]()

#### [Invoking decoders]()

Each new decoder (input\_entry) now provides a get\_guid() and get\_name() to allow user to choose the order in which they're invoked as well as disable individual installed decoders. Because of this, you are no longer supposed to walk input\_entry instances in your code; however many existing components do so because this was the way things were expected to work in all past versions before 1.4.

In most cases there's nothing that you need to do about this, unless you have code that talks to input\_entry instance methods directly.

##### [input\_manager]()

The proper way to instantiate any input related classes is to call input\_manager; it manages opening of decoders respecting user's decoder merit settings. Calling any of the helper methods to open decoders / read tags / etc in the SDK will call input\_manager if available (v1.4) and fall back to walking input\_entry services if not (v1.3).

##### [input\_entry shim]()

If your component targets API level lower than 79, all your attempts to walk input\_entry services return a shim service that redirects all your calls to input\_manager. You cannot walk actual input\_entry services.

This is to keep existing components working as intended.

If your component targets API level 79 (which means it won't load on v1.3), the shim is not installed as your component is expected to be aware of the new semantics.

#### [Implementing decoders]()

Many new input methods have been added and some are mandatory for all code, in particular reporting of decoder name and GUID; existing code not providing these will not compile. However some of the methods now required to be provided by an input class are mundane and will be left blank in majority of implementations; an input\_stubs class has been introduced to help with these. In past SDK versions, your input class would not derive from another class; it is now recommended to derive from input\_stubs to avoid having to supply mundane methods yourself.

##### [Specify supported interfaces]()

You can now control which level of decoder API your instance supports from your input class instead of using multi parameter input\_factory classes.

The input\_stubs class provides these for your convenience:

    typedef input_decoder_v4 interface_decoder_t;
    typedef input_info_reader interface_info_reader_t;
    typedef input_info_writer interface_info_writer_t;

Override these in your input class to indicate supported interfaces.

For an example, if your input supports remove\_tags(), indicate that you implement input\_info\_writer\_v2:

    typedef input_info_writer_v2 interface_info_writer_t;

<!-- SECTION "Decoders" [5127-7762] -->

### [Dynamic runtime]()

As of version 1.4, foobar2000 is compiled with dynamic Visual C runtime and redistributes Visual C runtime libraries with the installer, putting them in the foobar2000 installation folder if necessary. The benefits of this are: \* Smaller component DLLs \* Increased limit of how many component DLLs can be loaded.

This SDK comes configured for dynamic runtime by default. If you wish to support foobar2000 versions older than 1.4, change to static runtime or make sure that your users have the runtime installed.

<!-- SECTION "Dynamic runtime" [7763-8303] -->

### [service\_query()]()

tl;dr if you don't know what this is about you probably don't care and your component isn't in any way affected by this.

The way service\_query() is implemented has been redesigned for performance reasons - the old way ( implemented per interface class via FB2K\_MAKE\_SERVICE\_INTERFACE macro ) resulted in every single class\_guid from the entire SDK being included in your DLL, due to strange behaviors of Microsoft linker.

The service\_query() function itself is now provided by service\_impl\_\* classes, so it's materialized only for services that your code implements and spawns.

The default implementation calls a newly added static method of handle\_service\_query(), implemented in service\_base to check against all supported interfaces by walking derived class chain.

If you wish to override it, create a function with a matching signature in your class and it will be called instead:

    class myClass : public service_base {
    public:
        static bool handle_service_query(service_ptr & out, const GUID & guid, myClass * in) {
            if ( guid == myClassGUID ) {
                out = in; return true;
            }
            return false;
        }
    };

#### [Multi-inheritance]()

The above change to service\_query() implementation rules obviously breaks any existing code where one class inherits from multiple service classes and supplies a custom service\_query().

While using multi inheritance is not recommended and very rarely done, a new template has been added to help with such cases: service\_multi\_inherit\<class1, class2>.

You can use it to avoid having to supply service\_query() code yourself and possibly change it if service\_query() semantics change again in the future.

<!-- SECTION "service_query()" [8304-10014] -->

## [Version 1.3 notes]()

foobar2000 version 1.3 uses different metadb behavior semantics than older versions, to greatly improve the performance of multithreaded operation by reducing the time spent within global metadb locks.

Any methods that:

*   Lock the metadb - database\_lock() etc
*   Retrieve direct pointers to metadb info - get\_info\_locked() style

.. are now marked deprecated and implemented only for backwards compatibility; they should not be used in any new code.

It is recommended that you change your existing code using these to obtain track information using new get\_info\_ref() style methods for much better performance as these methods have minimal overhead and require no special care when used in multiple concurrent threads.

<!-- SECTION "Version 1.3 notes" [10015-10770] -->

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
*   Necessary headers from libPPUI and helpers, which both contain various code commonly used by fb2k components.

<!-- SECTION "Basic usage" [10771-12139] -->

## [Structure of a component]()

A component is a DLL that implements one or more entrypoint services and interacts with services provided by other components.

<!-- SECTION "Structure of a component" [12140-12306] -->

### [Services]()

A service type is an interface class, deriving directly or indirectly from `service_base` class. A service type class must not have any data members; it can only provide virtual methods (to be overridden by service implementation), helper non-virtual methods built around virtual methods, static helper methods, and constants / enums. Each service interface class must have a static `class_guid` member, used for identification when enumerating services or querying for supported functionality. A service type declaration should declare a class with public virtual/helper/static methods, and use `FB2K_MAKE_SERVICE_INTERFACE()` / `FB2K_MAKE_SERVICE_INTERFACE_ENTRYPOINT()` macro to implement standard service behaviors for the class; additionally, `class_guid` needs to be defined outside class declaration (e.g. `const GUID someclass::class_guid = {….};` ). Note that most of components will not declare their own service types, they will only implement existing ones declared in the SDK.

A service implementation is a class derived from relevant service type class, implementing virtual methods declared by service type class. Note that service implementation class does not implement virtual methods declared by service\_base; those are implemented by service type declaration framework (`service_query`) or by instantiation framework (`service_add_ref` / `service_release`). Service implementation classes are instantiated using `service_factory` templates in case of entrypoint services (see below), or using `service_impl_t` template and operator `new`: ”`myserviceptr = new service_impl_t<myservice_impl>(params);`”.

Each service object provides reference counter features and (`service_add_ref()` and `service_release()` methods) as well as a method to query for extended functionality (`service_query()` method). Those methods are implemented by service framework and should be never overridden by service implementations. These methods should also never be called directly - reference counter methods are managed by relevant autopointer templates, `service_query_t` function template should be used instead of calling `service_query` directly, to ensure type safety and correct type conversions.

<!-- SECTION "Services" [12307-14572] -->

### [Entrypoint services]()

An entrypoint service type is a special case of a service type that can be registered using `service_factory` templates, and then accessed from any point of service system (excluding DLL startup/shutdown code, such as code inside static object constructors/destructors). An entrypoint service type class must directly derive from `service_base`.

Registered entrypoint services can be accessed using:

*   For services types with variable number of implementations registered: `service_enum_t<T>` template, `service_class_helper_t<T>` template, etc, e.g.

        service_enum_t<someclass> e; service_ptr_t<someclass> ptr; while(e.next(ptr)) ptr->dosomething();

*   For services types with a single always-present implementation registered - such as core services like `playlist_manager` - using `someclass::get()`, e.g.:

        auto api = someclass::get(); api->dosomething(); api->dosomethingelse();

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

<!-- SECTION "Entrypoint services" [14573-17030] -->

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

<!-- SECTION "Service extensions" [17031-17970] -->

### [Autopointer template use]()

When performing most kinds of service operations, `service_ptr_t<T>` template should be used rather than working with service pointers directly; it automatically manages reference counter calls, ensuring that the service object is deleted when it is no longer referenced.

For convenience, all service classes have `myclass::ptr` typedef'd to `service_ptr_t<myclass>`.

When working with pointers to core fb2k services, just use C++11 `auto` keyword and `someclass::get()`, e.g. `auto myAPI = playlist_manager::get();`

<!-- SECTION "Autopointer template use" [17971-18538] -->

### [Exception use]()

Most of API functions use C++ exceptions to signal failure conditions. All used exception classes must derive from `std::exception` (which `pfc::exception` is typedef'd to); this design allows various instances of code to use single `catch()` line to get human-readable description of the problem to display.

Additionally, special subclasses of exceptions are defined for use in specific conditions, such as `exception_io` for I/O failures. As a result, you must provide an exception handler whenever you invoke any kind of I/O code that may fail, unless in specific case calling context already handles exceptions (e.g. input implementation code - any exceptions should be forwarded to calling context, since exceptions are a part of input API).

Implementations of global callback services such as `playlist_callback`, `playback_callback` or `library_callback` must not throw unhandled exceptions; behaviors in case they do are undefined (app termination is to be expected).

<!-- SECTION "Exception use" [18539-19556] -->

### [Storing configuration]()

In order to create your entries in the configuration file, you must instantiate some objects that derive from `cfg_var` class. Those can be either predefined classes (`cfg_int`, `cfg_string`, etc) or your own classes implementing relevant methods.

Each `cfg_var` instance has a GUID assigned, to identify its configuration file entry. The GUID is passed to its constructor (which implementations must take care of, typically by providing a constructor that takes a GUID and forwards it to `cfg_var` constructor).

Note that `cfg_var` objects can only be instantiated statically (either directly as static objects, or as members of other static objects). Additionally, you can create configuration data objects that can be accessed by other components, by implementing `config_object` service. Some standard configuration variables can be also accessed using `config_object` interface.

<!-- SECTION "Storing configuration" [19557-20492] -->

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

<!-- SECTION "Use of global callback services" [20493-22914] -->

## [Service class design guidelines (advanced)]()

This chapter describes things you should keep on your mind when designing your own service type classes. Since 99% of components will only implement existing service types rather than adding their own cross-DLL-communication protocols, you can probably skip reading this chapter.

<!-- SECTION "Service class design guidelines (advanced)" [22915-23251] -->

### [Cross-DLL safety]()

It is important that all function parameters used by virtual methods of services are cross-DLL safe (do not depend on compiler-specific or runtime-specific behaviors, so no unexpected behaviors occur when calling code is built with different compiler/runtime than callee). To achieve this, any classes passed around must be either simple objects with no structure that could possibly vary with different compilers/runtimes (i.e. make sure that any memory blocks are freed on the side that allocated them); easiest way to achieve this is to reduce all complex data objects or classes passed around to interfaces with virtual methods, with implementation details hidden from callee. For an example, use `pfc::string_base&` as parameter to a function that is meant to return variable-length strings.

<!-- SECTION "Cross-DLL safety" [23252-24079] -->

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

<!-- SECTION "Entrypoint service efficiency" [24080-] -->
