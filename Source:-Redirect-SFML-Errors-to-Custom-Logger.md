SFML provides verbose error output through `sf::err()` which by default is piped into `std::cerr`, but it can easily be redirect to a different `std::ostream`. What is not as easy to achieve, is pushing the output to a different logging class entierly.

The following class implements a `std::basic_streambuf` allowing to redirect the SFML error output to itself and then pushing the data to a logger. For this example `spdlog::logger` has been used, but can be easily replaced with a different logger or logging framework.

```cpp
auto logger = spdlog::stdout_color_mt("console"); // Example logger
auto log_redirection = StreamLogger{ sf::err(), logger };
```

As long as the `StreamLogger` instance exists, the SFML error output is redirected to the logger.

```cpp
#include <streambuf>
#include <ostream>

#include "spdlog/spdlog.h"

template<class Element = char, class Trait = std::char_traits<Element>>
class StreamLogger : public std::basic_streambuf<Element, Trait>
{
public:
    StreamLogger(std::ostream& stream, std::shared_ptr<spdlog::logger> logger)
        : m_stream{ stream }, m_logger{ std::move(logger) }
    {
        // Redirect provided stream
        m_buffer = m_stream.rdbuf(this);
    };

    ~StreamLogger()
    {
        try
        {
            // Restore provided stream
            m_stream.rdbuf(m_buffer);

        }
        catch(const std::ios_base::failure& exception)
        {
            const auto message = std::string{ "StreamLogger was unable to restore the provided stream: " };
            m_logger->error(message + exception.what());
        }
    }

    StreamLogger(const StreamLogger& other) = delete;
    StreamLogger& operator=(const StreamLogger& other) = delete;

    StreamLogger(StreamLogger&& other) noexcept = default;
    StreamLogger& operator=(StreamLogger&& other) noexcept = default;

protected:
    std::streamsize xsputn(const Element* elements, const std::streamsize count) override
    {
        m_string_buffer.append(elements, static_cast<std::string::size_type>(count));
        return count;
    }

    typename Trait::int_type overflow(typename Trait::int_type final_character) override
    {
        m_logger->error(m_string_buffer);
        m_string_buffer.clear();
        return Trait::not_eof(final_character);
    }

    std::basic_ostream<Element, Trait>& m_stream;
    std::streambuf* m_buffer;
    std::basic_string<Element, Trait> m_string_buffer;
    std::shared_ptr<spdlog::logger> m_logger;
};
```

### License

This is free and unencumbered software released into the public domain.