# 1. install vcpkg in windows

```cmd
> git clone https://github.com/microsoft/vcpkg
> .\vcpkg\bootstrap-vcpkg.bat
```

# 2. set vcpkg mirrors

```cmd
# set env
X_VCPKG_ASSET_SOURCES=x-azurl,http://106.15.181.5/
```

# 3. install cpprest

```cmd
vcpkg integrate install
vcpkg install cpprestsdk cpprestsdk:x64-windows
```

# 4. 使用cpprest实现异步流

```cpp
#include <iostream>
#include <utility>
#ifdef _WIN32
#include <fcntl.h>
#include <io.h>
#endif
#include <stddef.h>
#include <cpprest/http_client.h>
#include <cpprest/filestream.h>

using namespace utility;
using namespace web::http;
using namespace web::http::client;
using namespace concurrency::streams;
using std::cerr;
using std::endl;

#ifdef _WIN32
#define tcout std::wcout
#else
#define tcout std::cout
#endif

auto get_headers(http_response resp)
{
	auto headers = resp.to_string();
	auto end = headers.find(U("\r\n\r\n"));
	if (end != string_t::npos) {
		headers.resize(end + 4);
	}
	return headers;
}

auto get_request(string_t uri)
{
	http_client client{ uri };
	auto request = client.request(methods::GET)
		.then([](http_response resp) {
		if (resp.status_code() == status_codes::OK) {
			tcout << U("Saving...\n");
			ostream fs;
			fstream::open_ostream(U("results.html"), std::ios_base::out | std::ios_base::trunc)
				.then([&fs, resp](ostream os) {
				fs = os;
				return resp.body().read_to_end(fs.streambuf());
					})
				.then([&fs](size_t size) {
						fs.close();
						tcout << size << U(" bytes saved\n");
					})
						.wait();
		}
		else {
			auto headers = get_headers(resp);
			tcout << headers;
			tcout << resp.extract_string().get();
		}
			});
	return request;
}

#ifdef _WIN32
int wmain(int argc, wchar_t* argv[])
#else
int main(int argc, char* argv[])
#endif
{
#ifdef _WIN32
	_setmode(_fileno(stdout), _O_WTEXT);
#endif

	if (argc != 2) {
		cerr << "A URL is needed\n";
		return 1;
	}

	try {
		auto request = get_request(argv[1]);
		request.wait();
	}
	catch (const std::exception& e) {
		cerr << "Error exception: " << e.what() << endl;
	}
}

```

## 5. 简单的Http Server

```cpp
#include <exception>
#include <iostream>
#include <map>
#include <string>
#ifdef _WIN32
#include <fcntl.h>
#include <io.h>
#endif
#include <cpprest/http_listener.h>
#include <cpprest/json.h>

using namespace std;
using namespace utility;
using namespace web;
using namespace web::http;
using namespace web::http::experimental::listener;

#ifdef _WIN32
#define tcout std::wcout
#else
#define tcout std::cout
#endif

void handle_get(http_request req)
{
	auto& uri = req.request_uri();
	if (uri.path() != U("/sayHi")) {
		req.reply(status_codes::NotFound);
		return;
	}

	tcout << uri::decode(uri.query()) << endl;

	auto query = uri::split_query(uri.query());
	auto it = query.find(U("name"));
	if (it == query.end()) {
		req.reply(status_codes::BadRequest, U("Missing query info"));
		return;
	}

	auto answer = json::value::object(true);
	answer[U("msg")] = json::value(string_t(U("Hi, ")) + uri::decode(it->second) + U("!"));
	
	req.reply(status_codes::OK, answer);
}

int main()
{
#ifdef _WIN32
	_setmode(_fileno(stdout), _O_WTEXT);
#endif

	http_listener listener(U("http://127.0.0.1:8008/"));
	listener.support(methods::GET, handle_get);

	try {
		listener.open().wait();

		tcout << "Listening. Press ENTER to exit.\n";
		string line;
		getline(cin, line);

		listener.close().wait();
	}
	catch (const exception& e) {
		cerr << e.what() << endl;
		return 1;
	}

	return 0;
}

```
