# FineRpc

FineArtRpc是一种适应于快速开发且使用简单的Rpc程序，使用它你几乎不需要写代码，就能实现一个数据库所有增删改查的远程Rpc调用，包括存储过程，视图以及各种数据表，当然你也能使用它执行其它非数据库应用比如物联网应用中调用硬件方法的远程调用等

1、最简单的使用(仅使用Rpc功能，不使用数据库)

1.1 新建一个供前后端共同调用的类库，BaseRpc.csproj，并安装Nuget包FineArtRpcCore 项目文件如下：
	
	<Project Sdk="Microsoft.NET.Sdk">

		<PropertyGroup>
			<TargetFramework>netstandard2.0</TargetFramework>
			<LangVersion>latest</LangVersion>
			<ImplicitUsings>enable</ImplicitUsings>
			<Nullable>enable</Nullable>
		</PropertyGroup>

		<ItemGroup>
			<PackageReference Include="FineArtRpcCore" Version="1.*" />
		</ItemGroup>

	</Project>

1.2 写一个可供前后端共享的Rpc接口以及Rpc实现，TestRpcService.cs如下：
	
	using FineArtBase.Interfaces;	
	namespace BaseRpc;
	
	public interface ITestRpcServer: IRpcService
	{
	    Task<string> SayHelloAsync(string name);
	}
	
	public class TestRpcServer : ITestRpcServer
	{
	    public async Task<string> SayHelloAsync(string name) => await Task.FromResult($"你好，{name}").ConfigureAwait(false);
	}
		
1.3 新建一个Asp.Net Core Web Api应用程序，去掉配置https前面的勾(如果你有域名证书可以勾上)，项目文件如下：
	
	<Project Sdk="Microsoft.NET.Sdk.Web">
	
	  <PropertyGroup>
	    <TargetFramework>net9.0</TargetFramework>
	    <Nullable>enable</Nullable>
	    <ImplicitUsings>enable</ImplicitUsings>
	  </PropertyGroup>
	
	  <ItemGroup>
	    <ProjectReference Include="..\BaseRpc\BaseRpc.csproj" />
	  </ItemGroup>
	
	</Project>

1.4 修改Program.cs文件如下:

	using BaseRpc;
	using FineArtBase.Utilities;
	using FineArtRpcCore.Utilities;
	using Microsoft.AspNetCore.Connections;
	using Microsoft.AspNetCore.Server.Kestrel.Transport.Sockets;	
	var builder = WebApplication.CreateBuilder(args);
	builder.Services
	    .AddSingleton<IConnectionListenerFactory, SocketTransportFactory>()
	    .AddScoped<ITestRpcServer, TestRpcServer>()
	    .AddRpcServices(sp => new RpcOptions(5739).AddRpcService(sp.GetRequiredService<ITestRpcServer>()));
	var app = builder.Build();
	app.Run();

	至此，Rpc服务器创建完成，其中端口号为5738。

1.5 客户端调用RPC，创建一个控制台应用程序ConsoleApp1.csproj，并引用BaseRpc.csproj，项目文件如下：
  
    <Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
      <OutputType>Exe</OutputType>
      <TargetFramework>net9.0</TargetFramework>
      <ImplicitUsings>enable</ImplicitUsings>
      <Nullable>enable</Nullable>
    </PropertyGroup>

    <ItemGroup>
      <ProjectReference Include="..\BaseRpc\BaseRpc.csproj" />
    </ItemGroup>
    </Project>

1.6 修改Program.cs文件如下:
    
    using BaseRpc;
    using FineArtRpcCore.Utilities;

    RpcHelper.SetTcp("localhost", 5739);

    Console.WriteLine(await RpcHelper.GetTcpAsync<ITestRpcServer, string>(async service => await service.SayHelloAsync("张三李四王五")));

    Console.ReadLine();****
    



