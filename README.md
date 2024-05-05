# NSubstitute.Context

Welcome to the documentation for NSubstitute.Context! NSubstitute.Context is a powerful library designed to provide automatic mock creation for constructor parameters. This functionality greatly simplifies the process of setting up test scenarios by automatically generating mock objects for the dependencies required by the class under test.

In software development, constructors play a fundamental role in object creation. They allow developers to instantiate objects with predefined initial states, facilitating the process of working with complex data structures and classes. However, in some cases, constructors can become cumbersome, especially when dealing with classes that require numerous parameters for initialization.

Recognizing this challenge, NSubstitute.Context offers a solution by providing a constructor with support for multiple parameters. Whether you're creating an instance of a class with a handful of essential attributes or configuring an object with an extensive range of options, our flexible constructor empowers you to do so with ease.

This documentation will guide you through the usage of the constructor and demonstrate how to leverage its capabilities effectively within the NSubstitute.Context library. Whether you're a seasoned developer or just starting your journey in software development, we aim to make the process of working with constructors as straightforward and intuitive as possible.

Let's dive in and explore how NSubstitute.Context can simplify object initialization, automate mock creation, and streamline your development workflow. This is a typical problem during testing. The SUT has a lot of constructor parameters that need to be mocked before use.

    internal class ServiceContext : IServiceContext
    {
        private IClipboard _clipboard;
        private IMessageService _messageService;
        private IOpenFileService _openFileService;
        private IOpenFolderService _openFolderService;
        private ISaveFileService _saveFileService;
    
        public ServiceContext(
            IMessageService messageService,
            IClipboard clipboard,
            IOpenFileService openFileService,
            ISaveFileService saveFileService,
            IOpenFolderService openFolderService)
        {
            _messageService = messageService ?? throw new ArgumentNullException(nameof(messageService));
            _clipboard = clipboard ?? throw new ArgumentNullException(nameof(clipboard));
            _openFileService = openFileService ?? throw new ArgumentNullException(nameof(openFileService));
            _saveFileService = saveFileService ?? throw new ArgumentNullException(nameof(saveFileService));
            _openFolderService = openFolderService ?? throw new ArgumentNullException(nameof(openFolderService));
        }
    
    }

The `Context.For<TSut>` allows a simple mock creation process by creating the constructor parameters automatically.

    [Test]
    public void Create_And_Use_Context()
    {
        // arrange
        var ctx = new ContextFor<ServiceContext>();
        var sut = ctx.BuildSut();
    
        // act
        var result = sut.GetSomething();
    
        // assert
        result.Should().NotBeNull();
    }

The created mocks can be accessed using the `For<TParameter>` method and the type of the parameter. The method returns the parameter so the results of the mock can be modified if necessary.

    [Test]
    public void Inject_Something_Into_A_Constructor_Parameter()
    {
        // arrange
        var ctx = new ContextFor<ServiceContext>();
        // the proxys are created by NSubstitute, so I can access them
        ctx.For<IClipboard>().GetData().Returns(new ClipboardData() { Text = "Hello World." });
        var sut = ctx.BuildSut();
    
        // act
        sut.PasteClipboard();
    
        // assert
        sut.Control.Text.Should().Be("Hello World.");
    }

In most cases, this is sufficient because usually an Interface is used only once. If not or the type is more generic like a string, the name of the parameter can be passed.

In very rare cases, an auto generated mock is not desired or impossible to automatically create. For these rare cases, the `Use<TParam>(TParam mock)` method can be used to inject the mock directly.

    [Test]
    public void Or_Use_An_Instance_Directly()
    {
        // arrange
        var ctx = new ContextFor<ServiceContext>();
        // or we use an instance of a mock, in case the implementation is more complex
        ctx.Use<IClipboard>(new ClipboardMock());
        var sut = ctx.BuildSut();
    
        // act
        sut.PasteClipboard();
    
        // assert
        sut.Control.Text.Should().Be("Hello World.");
    }

        



