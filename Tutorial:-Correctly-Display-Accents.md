# Correctly display accents

To solve the problem of accents that are not displayed correctly, there are three steps:

1 - The encoding of pages .cpp must be UTF-8

2 - Put the text with an L in front (ex sf:: String (L"héhé"); )

3 - Using std::wcout or sf::String