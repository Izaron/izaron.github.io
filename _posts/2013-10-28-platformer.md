---
title: Creating a platformer game in 30 minutes
date: 2013-10-28 15:28:00
categories: [Longread]
---

*This post is the translation of [my blog bost from habr.com](https://habr.com/post/199064/).*

*The post is actual as for 2013 and contains some unavoidable cyrillic letters on screenshots. Sorry for inconvenience.*

Hello! Today we will make a platformer game, using **C++**, **Box2D** and **SFML**, as well as an editor of 2D game maps - **Tiled Map Editor**.

![](/assets/img/posts/2013-10-28/header.png)

Here is the result (the map was created in 5 minutes + the game was working slow during the recording +
the screen size isn't such stretched - it is a Bandicam defect):

{% include embed/youtube.html id='0NUAUMjQigI' %}

Source codes and a .exe file - at the end of the article.

# "What, where, when?"

## Box2D

We will use this library to simulate physics in the platformer (colliding with blocks, gravitation).
Maybe it was too much to use the library just for blocks, but let's have a fancy life. Why Box2D? Because it's the most popular and free physical library.

## SFML

Why SFML? I wanted to use SDL at first, but it's too limited in comparison to SFML, I'd had to invent some wheels by myself.
Just thanks to the author of SFML for saved time! We use it to draw graphics.

## Tiled Map Editor

What does Tiled Map Editor do here? Have you ever tried to create maps for games? I'd bet your very first map looked as something like this.

<details>
  <summary>Spoiler</summary>
<div class="language-plaintext highlighter-rouge">
<div class="highlight"><code><td class="rouge-code"><pre>
    1111111
    1000001
    1001001
    1000011
    1000111
    1111111</pre></td></code></div></div>
</details>

It's a pretty uneffective solution! It's better to write something like map editor, but we all understand that
it is a term constrution and harder than creating such "maps" as above.

Tiled Map Editor is one of such map editors. This is great and dope, and we can save every
map created in the editor (consists of objects, tiles, its layers) in a XML-like .tmx file and then parse it via the special library in C++. Let's dive in.

## Creating a map

Download TME from the official [site](http://www.mapeditor.org/). Create new map "File-\>Create..."

![](/assets/img/posts/2013-10-28/new-map.png)

The orientation should be orthogonal (unless you make an isometric platformer), and the layer format XML,
since we'll be reading this format. By the way, you can't change neither the layer format, nor the tile size in a created map.

### Tiles

Then we go to "Map-\>New tileset...", load our tileset:

![](/assets/img/posts/2013-10-28/load-tiles.png)

And now you must have something like this:

![](/assets/img/posts/2013-10-28/layers.png)

What is the matter of the tile layers? Almost every game has multi-layer maps.
The first layer is earth (ice, grass, etc.), the second one are buildings
(barracks, forts, city hall, etc., the background is transpared, by the way),
the third one are trees (spruce, fir, etc., the background the same). That is, we draw the first layer firstly, then the second, and the third at last.

The process of creating layers is imprinted on the next 3 screenshots (the screenshots are clickable):

![](/assets/img/posts/2013-10-28/layer1.png){: .w-25 .normal}
![](/assets/img/posts/2013-10-28/layer2.png){: .w-25 .normal}
![](/assets/img/posts/2013-10-28/layer3.png){: .w-25 .normal}

The list of the layers:

![](/assets/img/posts/2013-10-28/layer-list.png)

### Objects

What's an object in terms of TME? An object has its name, type and value properties. This panel is responsible for objects

![](/assets/img/posts/2013-10-28/panel.png)

You can learn the meaning of every button by yourself.
Then let's try to create an object.
Delete the layer named "Колобоша" ("Kolobosha"), create an object layer, you can use this name "Колобоша" again.
Choose "Insert tile object" from the object panel (or you can choose a Shape to place),
click on Kolobosha tile and place an object somewhere. Then click right mouse button on the object and then "Object properties...". Change the object name to Kolobosha.

Then save the map.

In general, there are not hard points in map editors. It's time to parse the map.

# Parsing maps

There is a cool library to parse XML files - [TinyXML](http://www.grinninglizard.com/tinyxml/), load its sources.

Create a Visual Studio project. Include TinyXML files (or just paste them to the project excluding xmltest.cpp).
Then link includies and lib-files of SFML in "Project->Properties". If you don't have a clue how to do this - welcome to Google.

Create `Level.h` for maps.
```c++
#ifndef LEVEL_H
#define LEVEL_H

#pragma comment(lib,"Box2D.lib")
#pragma comment(lib,"sfml-graphics.lib")
#pragma comment(lib,"sfml-window.lib")
#pragma comment(lib,"sfml-system.lib")

#include <string>
#include <vector>
#include <map>
#include <SFML/Graphics.hpp>
```

This is the beginning of the file.

Further we define the object structure.
```c++
struct Object
{
    int GetPropertyInt(std::string name);
    float GetPropertyFloat(std::string name);
    std::string GetPropertyString(std::string name);

    std::string name;
    std::string type;

    sf::Rect<int> rect;
    std::map<std::string, std::string> properties;

    sf::Sprite sprite;
};
```

Look at this. As I was saying, in TME every object may have properties.
Properties are taken from a XML file, written to `std::map`, and then in can
be obtained via the function `GetProperty(...)`. `name` is the object name, `type` is its type, `rect` is the rectangle
describing the object. At last, `sprite` is a sprite (an image) - is part of the tileset taken for the object. An object may don't have sprite.

Then we have the layer structure which is dead simple.

```c++
struct Layer
{
    int opacity;
    std::vector<sf::Sprite> tiles;
};
```

The layer has opacity (yeah, we can do a semitransparent layer!) and the list of tiles.

Then we have the level class:

<details>
  <summary>bool Level::LoadFromFile(std::string filename)</summary>
<div class="language-c++ highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
136
137
138
139
140
141
142
143
144
145
146
147
148
149
150
151
152
153
154
155
156
157
158
159
160
161
162
163
164
165
166
167
168
169
170
171
172
173
174
175
176
177
178
179
180
181
182
183
184
185
186
187
188
189
190
191
192
193
194
195
196
197
198
199
200
201
202
203
204
205
206
207
208
209
210
211
212
213
214
215
216
217
218
219
220
221
222
223
224
225
226
227
228
229
230
231
232
233
234
235
236
237
238
239
240
241
242
243
244
</pre></td><td class="rouge-code"><pre><span class="kt">bool</span> <span class="n">Level</span><span class="o">::</span><span class="n">LoadFromFile</span><span class="p">(</span><span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">filename</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">TiXmlDocument</span> <span class="n">levelFile</span><span class="p">(</span><span class="n">filename</span><span class="p">.</span><span class="n">c_str</span><span class="p">());</span>

    <span class="c1">// Load an XML-map</span>
    <span class="k">if</span><span class="p">(</span><span class="o">!</span><span class="n">levelFile</span><span class="p">.</span><span class="n">LoadFile</span><span class="p">())</span>
    <span class="p">{</span>
        <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"Loading level </span><span class="se">\"</span><span class="s">"</span> <span class="o">&lt;&lt;</span> <span class="n">filename</span> <span class="o">&lt;&lt;</span> <span class="s">"</span><span class="se">\"</span><span class="s"> failed."</span> <span class="o">&lt;&lt;</span> <span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>
        <span class="k">return</span> <span class="nb">false</span><span class="p">;</span>

    <span class="p">}</span>

    <span class="c1">// Use map container</span>

    <span class="n">TiXmlElement</span> <span class="o">*</span><span class="n">map</span><span class="p">;</span>
    <span class="n">map</span> <span class="o">=</span> <span class="n">levelFile</span><span class="p">.</span><span class="n">FirstChildElement</span><span class="p">(</span><span class="s">"map"</span><span class="p">);</span>

    <span class="c1">// An example of the map: &lt;map version="1.0" orientation="orthogonal"</span>
    <span class="c1">// width="10" height="10" tilewidth="34" tileheight="34"&gt;</span>
    <span class="n">width</span> <span class="o">=</span> <span class="n">atoi</span><span class="p">(</span><span class="n">map</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"width"</span><span class="p">));</span>
    <span class="n">height</span> <span class="o">=</span> <span class="n">atoi</span><span class="p">(</span><span class="n">map</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"height"</span><span class="p">));</span>
    <span class="n">tileWidth</span> <span class="o">=</span> <span class="n">atoi</span><span class="p">(</span><span class="n">map</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"tilewidth"</span><span class="p">));</span>
    <span class="n">tileHeight</span> <span class="o">=</span> <span class="n">atoi</span><span class="p">(</span><span class="n">map</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"tileheight"</span><span class="p">));</span>

    <span class="c1">// Take the tileset description and the id of the first tile</span>
    <span class="n">TiXmlElement</span> <span class="o">*</span><span class="n">tilesetElement</span><span class="p">;</span>
    <span class="n">tilesetElement</span> <span class="o">=</span> <span class="n">map</span><span class="o">-&gt;</span><span class="n">FirstChildElement</span><span class="p">(</span><span class="s">"tileset"</span><span class="p">);</span>
    <span class="n">firstTileID</span> <span class="o">=</span> <span class="n">atoi</span><span class="p">(</span><span class="n">tilesetElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"firstgid"</span><span class="p">));</span>

    <span class="c1">// source is the path to the image</span>
    <span class="n">TiXmlElement</span> <span class="o">*</span><span class="n">image</span><span class="p">;</span>
    <span class="n">image</span> <span class="o">=</span> <span class="n">tilesetElement</span><span class="o">-&gt;</span><span class="n">FirstChildElement</span><span class="p">(</span><span class="s">"image"</span><span class="p">);</span>
    <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">imagepath</span> <span class="o">=</span> <span class="n">image</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"source"</span><span class="p">);</span>

    <span class="c1">// Try to load the tileset</span>
    <span class="n">sf</span><span class="o">::</span><span class="n">Image</span> <span class="n">img</span><span class="p">;</span>

    <span class="k">if</span><span class="p">(</span><span class="o">!</span><span class="n">img</span><span class="p">.</span><span class="n">loadFromFile</span><span class="p">(</span><span class="n">imagepath</span><span class="p">))</span>
    <span class="p">{</span>
        <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"Failed to load tile sheet."</span> <span class="o">&lt;&lt;</span> <span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>
        <span class="k">return</span> <span class="nb">false</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="c1">// Clear the color (109, 159, 185) out of the map</span>
    <span class="c1">// In general, the tileset can have any color as background, but I didn't find a solution</span>
    <span class="c1">// to parse a hex string like "6d9fb9" to the color</span>
    <span class="n">img</span><span class="p">.</span><span class="n">createMaskFromColor</span><span class="p">(</span><span class="n">sf</span><span class="o">::</span><span class="n">Color</span><span class="p">(</span><span class="mi">109</span><span class="p">,</span> <span class="mi">159</span><span class="p">,</span> <span class="mi">185</span><span class="p">));</span>
    <span class="c1">// Load the texture from the image</span>
    <span class="n">tilesetImage</span><span class="p">.</span><span class="n">loadFromImage</span><span class="p">(</span><span class="n">img</span><span class="p">);</span>
    <span class="c1">// Smooth is prohibited</span>
    <span class="n">tilesetImage</span><span class="p">.</span><span class="n">setSmooth</span><span class="p">(</span><span class="nb">false</span><span class="p">);</span>

    <span class="c1">// Get the amount of tileset rows and columns</span>
    <span class="kt">int</span> <span class="n">columns</span> <span class="o">=</span> <span class="n">tilesetImage</span><span class="p">.</span><span class="n">getSize</span><span class="p">().</span><span class="n">x</span> <span class="o">/</span> <span class="n">tileWidth</span><span class="p">;</span>
    <span class="kt">int</span> <span class="n">rows</span> <span class="o">=</span> <span class="n">tilesetImage</span><span class="p">.</span><span class="n">getSize</span><span class="p">().</span><span class="n">y</span> <span class="o">/</span> <span class="n">tileHeight</span><span class="p">;</span>

    <span class="c1">// A vector consists of image rectangles (TextureRect)</span>
    <span class="n">std</span><span class="o">::</span><span class="n">vector</span><span class="o">&lt;</span><span class="n">sf</span><span class="o">::</span><span class="n">Rect</span><span class="o">&lt;</span><span class="kt">int</span><span class="o">&gt;&gt;</span> <span class="n">subRects</span><span class="p">;</span>

    <span class="k">for</span><span class="p">(</span><span class="kt">int</span> <span class="n">y</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">y</span> <span class="o">&lt;</span> <span class="n">rows</span><span class="p">;</span> <span class="n">y</span><span class="o">++</span><span class="p">)</span>
    <span class="k">for</span><span class="p">(</span><span class="kt">int</span> <span class="n">x</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">x</span> <span class="o">&lt;</span> <span class="n">columns</span><span class="p">;</span> <span class="n">x</span><span class="o">++</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">sf</span><span class="o">::</span><span class="n">Rect</span><span class="o">&lt;</span><span class="kt">int</span><span class="o">&gt;</span> <span class="n">rect</span><span class="p">;</span>

        <span class="n">rect</span><span class="p">.</span><span class="n">top</span> <span class="o">=</span> <span class="n">y</span> <span class="o">*</span> <span class="n">tileHeight</span><span class="p">;</span>
        <span class="n">rect</span><span class="p">.</span><span class="n">height</span> <span class="o">=</span> <span class="n">tileHeight</span><span class="p">;</span>
        <span class="n">rect</span><span class="p">.</span><span class="n">left</span> <span class="o">=</span> <span class="n">x</span> <span class="o">*</span> <span class="n">tileWidth</span><span class="p">;</span>
        <span class="n">rect</span><span class="p">.</span><span class="n">width</span> <span class="o">=</span> <span class="n">tileWidth</span><span class="p">;</span>

        <span class="n">subRects</span><span class="p">.</span><span class="n">push_back</span><span class="p">(</span><span class="n">rect</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="c1">// Working with layers</span>
    <span class="n">TiXmlElement</span> <span class="o">*</span><span class="n">layerElement</span><span class="p">;</span>
    <span class="n">layerElement</span> <span class="o">=</span> <span class="n">map</span><span class="o">-&gt;</span><span class="n">FirstChildElement</span><span class="p">(</span><span class="s">"layer"</span><span class="p">);</span>
    <span class="k">while</span><span class="p">(</span><span class="n">layerElement</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">Layer</span> <span class="n">layer</span><span class="p">;</span>
		
        <span class="c1">// If we have the opacity property, then set the opacity, otherwise it won't be semitransparent</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">layerElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"opacity"</span><span class="p">)</span> <span class="o">!=</span> <span class="nb">NULL</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="kt">float</span> <span class="n">opacity</span> <span class="o">=</span> <span class="n">strtod</span><span class="p">(</span><span class="n">layerElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"opacity"</span><span class="p">),</span> <span class="nb">NULL</span><span class="p">);</span>
            <span class="n">layer</span><span class="p">.</span><span class="n">opacity</span> <span class="o">=</span> <span class="mi">255</span> <span class="o">*</span> <span class="n">opacity</span><span class="p">;</span>
        <span class="p">}</span>
        <span class="k">else</span>
        <span class="p">{</span>
            <span class="n">layer</span><span class="p">.</span><span class="n">opacity</span> <span class="o">=</span> <span class="mi">255</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="c1">// &lt;data&gt; container</span>
        <span class="n">TiXmlElement</span> <span class="o">*</span><span class="n">layerDataElement</span><span class="p">;</span>
        <span class="n">layerDataElement</span> <span class="o">=</span> <span class="n">layerElement</span><span class="o">-&gt;</span><span class="n">FirstChildElement</span><span class="p">(</span><span class="s">"data"</span><span class="p">);</span>

        <span class="k">if</span><span class="p">(</span><span class="n">layerDataElement</span> <span class="o">==</span> <span class="nb">NULL</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"Bad map. No layer information found."</span> <span class="o">&lt;&lt;</span> <span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="c1">// &lt;tile&gt; container - description of tiles of each layer</span>
        <span class="n">TiXmlElement</span> <span class="o">*</span><span class="n">tileElement</span><span class="p">;</span>
        <span class="n">tileElement</span> <span class="o">=</span> <span class="n">layerDataElement</span><span class="o">-&gt;</span><span class="n">FirstChildElement</span><span class="p">(</span><span class="s">"tile"</span><span class="p">);</span>

        <span class="k">if</span><span class="p">(</span><span class="n">tileElement</span> <span class="o">==</span> <span class="nb">NULL</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"Bad map. No tile information found."</span> <span class="o">&lt;&lt;</span> <span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>
            <span class="k">return</span> <span class="nb">false</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="kt">int</span> <span class="n">x</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
        <span class="kt">int</span> <span class="n">y</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>

        <span class="k">while</span><span class="p">(</span><span class="n">tileElement</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="kt">int</span> <span class="n">tileGID</span> <span class="o">=</span> <span class="n">atoi</span><span class="p">(</span><span class="n">tileElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"gid"</span><span class="p">));</span>
            <span class="kt">int</span> <span class="n">subRectToUse</span> <span class="o">=</span> <span class="n">tileGID</span> <span class="o">-</span> <span class="n">firstTileID</span><span class="p">;</span>

            <span class="c1">// Set the TextureRect of each tile</span>
            <span class="k">if</span> <span class="p">(</span><span class="n">subRectToUse</span> <span class="o">&gt;=</span> <span class="mi">0</span><span class="p">)</span>
            <span class="p">{</span>
                <span class="n">sf</span><span class="o">::</span><span class="n">Sprite</span> <span class="n">sprite</span><span class="p">;</span>
                <span class="n">sprite</span><span class="p">.</span><span class="n">setTexture</span><span class="p">(</span><span class="n">tilesetImage</span><span class="p">);</span>
                <span class="n">sprite</span><span class="p">.</span><span class="n">setTextureRect</span><span class="p">(</span><span class="n">subRects</span><span class="p">[</span><span class="n">subRectToUse</span><span class="p">]);</span>
                <span class="n">sprite</span><span class="p">.</span><span class="n">setPosition</span><span class="p">(</span><span class="n">x</span> <span class="o">*</span> <span class="n">tileWidth</span><span class="p">,</span> <span class="n">y</span> <span class="o">*</span> <span class="n">tileHeight</span><span class="p">);</span>
                <span class="n">sprite</span><span class="p">.</span><span class="n">setColor</span><span class="p">(</span><span class="n">sf</span><span class="o">::</span><span class="n">Color</span><span class="p">(</span><span class="mi">255</span><span class="p">,</span> <span class="mi">255</span><span class="p">,</span> <span class="mi">255</span><span class="p">,</span> <span class="n">layer</span><span class="p">.</span><span class="n">opacity</span><span class="p">));</span>

                <span class="n">layer</span><span class="p">.</span><span class="n">tiles</span><span class="p">.</span><span class="n">push_back</span><span class="p">(</span><span class="n">sprite</span><span class="p">);</span>
            <span class="p">}</span>

            <span class="n">tileElement</span> <span class="o">=</span> <span class="n">tileElement</span><span class="o">-&gt;</span><span class="n">NextSiblingElement</span><span class="p">(</span><span class="s">"tile"</span><span class="p">);</span>

            <span class="n">x</span><span class="o">++</span><span class="p">;</span>
            <span class="k">if</span> <span class="p">(</span><span class="n">x</span> <span class="o">&gt;=</span> <span class="n">width</span><span class="p">)</span>
            <span class="p">{</span>
                <span class="n">x</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
                <span class="n">y</span><span class="o">++</span><span class="p">;</span>
                <span class="k">if</span><span class="p">(</span><span class="n">y</span> <span class="o">&gt;=</span> <span class="n">height</span><span class="p">)</span>
                    <span class="n">y</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
            <span class="p">}</span>
        <span class="p">}</span>

        <span class="n">layers</span><span class="p">.</span><span class="n">push_back</span><span class="p">(</span><span class="n">layer</span><span class="p">);</span>

        <span class="n">layerElement</span> <span class="o">=</span> <span class="n">layerElement</span><span class="o">-&gt;</span><span class="n">NextSiblingElement</span><span class="p">(</span><span class="s">"layer"</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="c1">// Working wih objects</span>
    <span class="n">TiXmlElement</span> <span class="o">*</span><span class="n">objectGroupElement</span><span class="p">;</span>

    <span class="c1">// If we have object layers</span>
    <span class="k">if</span> <span class="p">(</span><span class="n">map</span><span class="o">-&gt;</span><span class="n">FirstChildElement</span><span class="p">(</span><span class="s">"objectgroup"</span><span class="p">)</span> <span class="o">!=</span> <span class="nb">NULL</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">objectGroupElement</span> <span class="o">=</span> <span class="n">map</span><span class="o">-&gt;</span><span class="n">FirstChildElement</span><span class="p">(</span><span class="s">"objectgroup"</span><span class="p">);</span>
        <span class="k">while</span> <span class="p">(</span><span class="n">objectGroupElement</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="c1">// &lt;object&gt; container</span>
            <span class="n">TiXmlElement</span> <span class="o">*</span><span class="n">objectElement</span><span class="p">;</span>
            <span class="n">objectElement</span> <span class="o">=</span> <span class="n">objectGroupElement</span><span class="o">-&gt;</span><span class="n">FirstChildElement</span><span class="p">(</span><span class="s">"object"</span><span class="p">);</span>

            <span class="k">while</span><span class="p">(</span><span class="n">objectElement</span><span class="p">)</span>
            <span class="p">{</span>
                <span class="c1">// Get all data - type, name, position, etc.</span>
                <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">objectType</span><span class="p">;</span>
                <span class="k">if</span> <span class="p">(</span><span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"type"</span><span class="p">)</span> <span class="o">!=</span> <span class="nb">NULL</span><span class="p">)</span>
                <span class="p">{</span>
                    <span class="n">objectType</span> <span class="o">=</span> <span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"type"</span><span class="p">);</span>
                <span class="p">}</span>
                <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">objectName</span><span class="p">;</span>
                <span class="k">if</span> <span class="p">(</span><span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"name"</span><span class="p">)</span> <span class="o">!=</span> <span class="nb">NULL</span><span class="p">)</span>

                <span class="p">{</span>
                    <span class="n">objectName</span> <span class="o">=</span> <span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"name"</span><span class="p">);</span>
                <span class="p">}</span>
                <span class="kt">int</span> <span class="n">x</span> <span class="o">=</span> <span class="n">atoi</span><span class="p">(</span><span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"x"</span><span class="p">));</span>
                <span class="kt">int</span> <span class="n">y</span> <span class="o">=</span> <span class="n">atoi</span><span class="p">(</span><span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"y"</span><span class="p">));</span>

                <span class="kt">int</span> <span class="n">width</span><span class="p">,</span> <span class="n">height</span><span class="p">;</span>

                <span class="n">sf</span><span class="o">::</span><span class="n">Sprite</span> <span class="n">sprite</span><span class="p">;</span>
                <span class="n">sprite</span><span class="p">.</span><span class="n">setTexture</span><span class="p">(</span><span class="n">tilesetImage</span><span class="p">);</span>
                <span class="n">sprite</span><span class="p">.</span><span class="n">setTextureRect</span><span class="p">(</span><span class="n">sf</span><span class="o">::</span><span class="n">Rect</span><span class="o">&lt;</span><span class="kt">int</span><span class="o">&gt;</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span><span class="mi">0</span><span class="p">,</span><span class="mi">0</span><span class="p">,</span><span class="mi">0</span><span class="p">));</span>
                <span class="n">sprite</span><span class="p">.</span><span class="n">setPosition</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">);</span>

                <span class="k">if</span> <span class="p">(</span><span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"width"</span><span class="p">)</span> <span class="o">!=</span> <span class="nb">NULL</span><span class="p">)</span>
                <span class="p">{</span>
                    <span class="n">width</span> <span class="o">=</span> <span class="n">atoi</span><span class="p">(</span><span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"width"</span><span class="p">));</span>
                    <span class="n">height</span> <span class="o">=</span> <span class="n">atoi</span><span class="p">(</span><span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"height"</span><span class="p">));</span>
                <span class="p">}</span>
                <span class="k">else</span>
                <span class="p">{</span>
                    <span class="n">width</span> <span class="o">=</span> <span class="n">subRects</span><span class="p">[</span><span class="n">atoi</span><span class="p">(</span><span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"gid"</span><span class="p">))</span> <span class="o">-</span> <span class="n">firstTileID</span><span class="p">].</span><span class="n">width</span><span class="p">;</span>
                    <span class="n">height</span> <span class="o">=</span> <span class="n">subRects</span><span class="p">[</span><span class="n">atoi</span><span class="p">(</span><span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"gid"</span><span class="p">))</span> <span class="o">-</span> <span class="n">firstTileID</span><span class="p">].</span><span class="n">height</span><span class="p">;</span>
                    <span class="n">sprite</span><span class="p">.</span><span class="n">setTextureRect</span><span class="p">(</span><span class="n">subRects</span><span class="p">[</span><span class="n">atoi</span><span class="p">(</span><span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"gid"</span><span class="p">))</span> <span class="o">-</span> <span class="n">firstTileID</span><span class="p">]);</span>
                <span class="p">}</span>

                <span class="c1">// Create object structure</span>
                <span class="n">Object</span> <span class="n">object</span><span class="p">;</span>
                <span class="n">object</span><span class="p">.</span><span class="n">name</span> <span class="o">=</span> <span class="n">objectName</span><span class="p">;</span>
                <span class="n">object</span><span class="p">.</span><span class="n">type</span> <span class="o">=</span> <span class="n">objectType</span><span class="p">;</span>
                <span class="n">object</span><span class="p">.</span><span class="n">sprite</span> <span class="o">=</span> <span class="n">sprite</span><span class="p">;</span>

                <span class="n">sf</span><span class="o">::</span><span class="n">Rect</span> <span class="o">&lt;</span><span class="kt">int</span><span class="o">&gt;</span> <span class="n">objectRect</span><span class="p">;</span>
                <span class="n">objectRect</span><span class="p">.</span><span class="n">top</span> <span class="o">=</span> <span class="n">y</span><span class="p">;</span>
                <span class="n">objectRect</span><span class="p">.</span><span class="n">left</span> <span class="o">=</span> <span class="n">x</span><span class="p">;</span>
                <span class="n">objectRect</span><span class="p">.</span><span class="n">height</span> <span class="o">=</span> <span class="n">height</span><span class="p">;</span>
                <span class="n">objectRect</span><span class="p">.</span><span class="n">width</span> <span class="o">=</span> <span class="n">width</span><span class="p">;</span>
                <span class="n">object</span><span class="p">.</span><span class="n">rect</span> <span class="o">=</span> <span class="n">objectRect</span><span class="p">;</span>

                <span class="c1">// Properties of the object</span>
                <span class="n">TiXmlElement</span> <span class="o">*</span><span class="n">properties</span><span class="p">;</span>
                <span class="n">properties</span> <span class="o">=</span> <span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">FirstChildElement</span><span class="p">(</span><span class="s">"properties"</span><span class="p">);</span>
                <span class="k">if</span> <span class="p">(</span><span class="n">properties</span> <span class="o">!=</span> <span class="nb">NULL</span><span class="p">)</span>
                <span class="p">{</span>
                    <span class="n">TiXmlElement</span> <span class="o">*</span><span class="n">prop</span><span class="p">;</span>
                    <span class="n">prop</span> <span class="o">=</span> <span class="n">properties</span><span class="o">-&gt;</span><span class="n">FirstChildElement</span><span class="p">(</span><span class="s">"property"</span><span class="p">);</span>
                    <span class="k">if</span> <span class="p">(</span><span class="n">prop</span> <span class="o">!=</span> <span class="nb">NULL</span><span class="p">)</span>
                    <span class="p">{</span>
                        <span class="k">while</span><span class="p">(</span><span class="n">prop</span><span class="p">)</span>
                        <span class="p">{</span>
                            <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">propertyName</span> <span class="o">=</span> <span class="n">prop</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"name"</span><span class="p">);</span>
                            <span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">propertyValue</span> <span class="o">=</span> <span class="n">prop</span><span class="o">-&gt;</span><span class="n">Attribute</span><span class="p">(</span><span class="s">"value"</span><span class="p">);</span>

                            <span class="n">object</span><span class="p">.</span><span class="n">properties</span><span class="p">[</span><span class="n">propertyName</span><span class="p">]</span> <span class="o">=</span> <span class="n">propertyValue</span><span class="p">;</span>

                            <span class="n">prop</span> <span class="o">=</span> <span class="n">prop</span><span class="o">-&gt;</span><span class="n">NextSiblingElement</span><span class="p">(</span><span class="s">"property"</span><span class="p">);</span>
                        <span class="p">}</span>
                    <span class="p">}</span>
                <span class="p">}</span>

                <span class="c1">// Pass the object to the vector</span>
                <span class="n">objects</span><span class="p">.</span><span class="n">push_back</span><span class="p">(</span><span class="n">object</span><span class="p">);</span>

                <span class="n">objectElement</span> <span class="o">=</span> <span class="n">objectElement</span><span class="o">-&gt;</span><span class="n">NextSiblingElement</span><span class="p">(</span><span class="s">"object"</span><span class="p">);</span>
            <span class="p">}</span>
            <span class="n">objectGroupElement</span> <span class="o">=</span> <span class="n">objectGroupElement</span><span class="o">-&gt;</span><span class="n">NextSiblingElement</span><span class="p">(</span><span class="s">"objectgroup"</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>
    <span class="k">else</span>
    <span class="p">{</span>
        <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"No object layers found..."</span> <span class="o">&lt;&lt;</span> <span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="nb">true</span><span class="p">;</span>
<span class="p">}</span>
</pre></td></tr></tbody></table></code></div></div>
</details>

The rest of Level functions:

```c++
Object Level::GetObject(std::string name)
{
    // Only the first object with given name
    for (int i = 0; i < objects.size(); i++)
        if (objects[i].name == name)
            return objects[i];
}

std::vector<Object> Level::GetObjects(std::string name)
{
    // All objects with given name
    std::vector<Object> vec;

    for(int i = 0; i < objects.size(); i++)
        if(objects[i].name == name)
            vec.push_back(objects[i]);

    return vec;
}

sf::Vector2i Level::GetTileSize()
{
    return sf::Vector2i(tileWidth, tileHeight);
}

void Level::Draw(sf::RenderWindow &window)
{
    // Draw all the tiles (DON'T draw the objects!)
    for(int layer = 0; layer < layers.size(); layer++)
        for(int tile = 0; tile < layers[layer].tiles.size(); tile++)
            window.draw(layers[layer].tiles[tile]);
}
```

We're done with `Level.h`!

Let's test it. Create `main.cpp` and write:

```c++
#include "level.h"

int main()
{
    Level level;
    level.LoadFromFile("test.tmx");

    sf::RenderWindow window;
    window.create(sf::VideoMode(800, 600), "Level.h test");

    while(window.isOpen())
    {
        sf::Event event;

        while(window.pollEvent(event))
        {
            if(event.type == sf::Event::Closed)
                window.close();
        }

        window.clear();

        level.Draw(window);
        window.display();
    }

    return 0;
}
```

The map can look as you want!

You can play with objects:

![](/assets/img/posts/2013-10-28/full.png)

`main.cpp`

```c++
#include "level.h"
#include <iostream>

int main()
{
    Level level;
    level.LoadFromFile("test.tmx");

    Object kolobosha = level.GetObject("Kolobosha");
    std::cout << kolobosha.name << std::endl;
    std::cout << kolobosha.type << std::endl;
    std::cout << kolobosha.GetPropertyInt("health") << std::endl;
    std::cout << kolobosha.GetPropertyString("mood") << std::endl;

    sf::RenderWindow window;
    window.create(sf::VideoMode(800, 600), "Kolobosha adventures");

    while(window.isOpen())
    {
        sf::Event event;

        while(window.pollEvent(event))
        {
            if(event.type == sf::Event::Closed)
                window.close();
        }

        window.clear();
        level.Draw(window);
        window.display();
    }

    return 0;
}
```

Results:

![](/assets/img/posts/2013-10-28/simple.png)

Then you're done with objects, it's time for Box2D:

# Falling boxes

We want to create a ~3D-action~ platformer game. There are objects on the map, namely player, enemy, block, or coins.
We initialize the player, force him to do some commands via keyboard and have physics laws.
Enemies walk here and there, push away the player if he's too close and die if the player jumped on them.
Blocks are fixed "in space" as static objects, the player can jump on them. Coins give nothing, just disapper when collided with the player.

Open `main.cpp`, erase all the code we wrote earlier, and write this:

```c++
#include "level.h"
#include <Box2D\Box2D.h>

#include <iostream>
#include <random>

Object player;
b2Body* playerBody;

std::vector<Object> coin;
std::vector<b2Body*> coinBody;

std::vector<Object> enemy;
std::vector<b2Body*> enemyBody;
```

Here we include `level.h` and `Box2D.h`. `iostream` is needed to output data to the console, random to generate
the direction of enemy movement. Then we have the player and vectors - each enemy, coin and the player have own Object and b2Body (a body in terms of Box2D).

Warning - blocks have no body, because they're active in the game only as physical objects, and don't take part in the game logic.

Further:

```c++
int main()
{
    srand(time(NULL));

    Level lvl;
    lvl.LoadFromFile("platformer.tmx");


    b2Vec2 gravity(0.0f, 1.0f);
    b2World world(gravity);

    sf::Vector2i tileSize = lvl.GetTileSize();
```

`srand(time(NULL))` is needed for random.

Load the map, create b2World, passing gravity to this. Also, the gravity may have another direction, and direction from
(0, 10) is stronger than (0, 1). Then we take the needed tile size.

Then we create block bodies:

```c++
    std::vector<Object> block = lvl.GetObjects("block");
    for(int i = 0; i < block.size(); i++)
    {
        b2BodyDef bodyDef;
        bodyDef.type = b2_staticBody;
        bodyDef.position.Set(block[i].rect.left + tileSize.x / 2 * (block[i].rect.width / tileSize.x - 1),
            block[i].rect.top + tileSize.y / 2 * (block[i].rect.height / tileSize.y - 1));
        b2Body* body = world.CreateBody(&bodyDef);
        b2PolygonShape shape;
        shape.SetAsBox(block[i].rect.width / 2, block[i].rect.height / 2);
        body->CreateFixture(&shape,1.0f);
    }
```

```c++
bodyDef.type = b2_staticBody;
```
Blocks are static, they have no weight and fixed in space:

```c++
bodyDef.position.Set(block[i].rect.left + tileSize.x / 2 * (block[i].rect.width / tileSize.x - 1),
    block[i].rect.top + tileSize.y / 2 * (block[i].rect.height / tileSize.y - 1));
```
Here we set block positions. If we set the same position as the object has, we will have a cruel error.

```c++
b2Body* body = world.CreateBody(&bodyDef);
```
Create the block body in `world`. We don't have any work with the body later (and don't save it anywhere).

```c++
b2PolygonShape shape;
shape.SetAsBox(block[i].rect.width / 2, block[i].rect.height / 2);
```
Each body has some shapes. I won't go deep to this theme, because the blocks (and other bodies) are happy with a single rectangle.

```c++
body->CreateFixture(&shape,1.0f);
```
Link the shape with the body.

Then we do the same with enemies, coins and the player, with slight differencies:

```c++
    coin = lvl.GetObjects("coin");
    for(int i = 0; i < coin.size(); i++)
    {
        b2BodyDef bodyDef;
        bodyDef.type = b2_dynamicBody;
        bodyDef.position.Set(coin[i].rect.left + tileSize.x / 2 * (coin[i].rect.width / tileSize.x - 1),
            coin[i].rect.top + tileSize.y / 2 * (coin[i].rect.height / tileSize.y - 1));
        bodyDef.fixedRotation = true;
        b2Body* body = world.CreateBody(&bodyDef);
        b2PolygonShape shape;
        shape.SetAsBox(coin[i].rect.width / 2, coin[i].rect.height / 2);
        body->CreateFixture(&shape,1.0f);
        coinBody.push_back(body);
    }

    enemy = lvl.GetObjects("enemy");
    for(int i = 0; i < enemy.size(); i++)
    {
        b2BodyDef bodyDef;
        bodyDef.type = b2_dynamicBody;
        bodyDef.position.Set(enemy[i].rect.left +
            tileSize.x / 2 * (enemy[i].rect.width / tileSize.x - 1),
            enemy[i].rect.top + tileSize.y / 2 * (enemy[i].rect.height / tileSize.y - 1));
        bodyDef.fixedRotation = true;
        b2Body* body = world.CreateBody(&bodyDef);

        b2PolygonShape shape;
        shape.SetAsBox(enemy[i].rect.width / 2, enemy[i].rect.height / 2);
        body->CreateFixture(&shape,1.0f);
        enemyBody.push_back(body);
    }


    player = lvl.GetObject("player");
    b2BodyDef bodyDef;
    bodyDef.type = b2_dynamicBody;
    bodyDef.position.Set(player.rect.left, player.rect.top);
    bodyDef.fixedRotation = true;
    playerBody = world.CreateBody(&bodyDef);
    b2PolygonShape shape; shape.SetAsBox(player.rect.width / 2, player.rect.height / 2);
    b2FixtureDef fixtureDef;
    fixtureDef.shape = &shape;
    fixtureDef.density = 1.0f; fixtureDef.friction = 0.3f;
    playerBody->CreateFixture(&fixtureDef);
```

```c++
bodyDef.fixedRotation = true;
```
This means that the body can't rotate.

All bodies done, it's time to initialize graphics!

```c++
    sf::Vector2i screenSize(800, 600);

    sf::RenderWindow window;
    window.create(sf::VideoMode(screenSize.x, screenSize.y), "Game");
```

Obvious code, creates a window with certain size and title.

```c++
    sf::View view;
    view.reset(sf::FloatRect(0.0f, 0.0f, screenSize.x, screenSize.y));
    view.setViewport(sf::FloatRect(0.0f, 0.0f, 2.0f, 2.0f));
```

Here we create a View for the window.

Why do we need this? To have a game in pixel art style, we multiply the screen size by 2 with using sf::View and all images are drawn with 2 times wider width and height.

```c++
while(window.isOpen())
{
    sf::Event evt;

    while(window.pollEvent(evt))
    {
        switch(evt.type)
        {
            case sf::Event::Closed:
                window.close();
                break;
```

The window closes by pressing the red cross. We had the same code earlier.

```c++
            case sf::Event::KeyPressed:
                if(evt.key.code == sf::Keyboard::W)
                    playerBody->SetLinearVelocity(b2Vec2(0.0f, -15.0f));

                if(evt.key.code == sf::Keyboard::D)
                    playerBody->SetLinearVelocity(b2Vec2(5.0f, 0.0f));

                if(evt.key.code == sf::Keyboard::A)
                    playerBody->SetLinearVelocity(b2Vec2(-5.0f, 0.0f));

                break;
```

This is more interesting. We add speed to the player by pressing WAD keys.

```c++
world.Step(1.0f / 60.0f, 1, 1);
```

We update the Box2D world here. The first argument takes the frequency of updating (every 1⁄60 seconds),
and the numbers of velocityIterations and positionIterations. The higher the last two values,
the more realistic physics the game has. Since we don't have any hard shapes like in Angry Birds, only rectangles, we're okay with a single iteration.

```c++
        for(b2ContactEdge* ce = playerBody->GetContactList(); ce; ce = ce->next) {
            b2Contact* c = ce->contact;
```
Here we work with collisions between the player and other bodies.

```c++
        for(int i = 0; i < coinBody.size(); i++)
            if(c->GetFixtureA() == coinBody[i]->GetFixtureList())
            {
                coinBody[i]->DestroyFixture(coinBody[i]->GetFixtureList());
                coin.erase(coin.begin() + i);
                coinBody.erase(coinBody.begin() + i);
            }
```

Collisions with coins. If a money collided with the player, it just removed from the vectors.

```c++
        for(int i = 0; i < enemyBody.size(); i++)
            if(c->GetFixtureA() == enemyBody[i]->GetFixtureList())
            {
                if(playerBody->GetPosition().y < enemyBody[i]->GetPosition().y)
                {
                    playerBody->SetLinearVelocity(b2Vec2(0.0f, -10.0f));

                    enemyBody[i]->DestroyFixture(enemyBody[i]->GetFixtureList());
                    enemy.erase(enemy.begin() + i);
                    enemyBody.erase(enemyBody.begin() + i);
                }
```

If an enemy collided with the player, we check if the player above the enemy or not. If yes, then the enemy removes, and the player springs up.

Otherwise, the player pushed away from the enemy.

```c++
                    else
                    {
                        int tmp = (playerBody->GetPosition().x < enemyBody[i]->GetPosition().x)
                            ? -1 : 1;
                        playerBody->SetLinearVelocity(b2Vec2(10.0f * tmp, 0.0f));
                    }
                }
        }
```

The player moves either to the left or to the right depending on his current position relatively to the enemy.

```c++
        for(int i = 0; i < enemyBody.size(); i++)
        {
            if(enemyBody[i]->GetLinearVelocity() == b2Vec2_zero)
            {
                int tmp = (rand() % 2 == 1) ? 1 : -1;
                enemyBody[i]->SetLinearVelocity(b2Vec2(5.0f * tmp, 0.0f));
            }
        }
```

If the speed of an enemy is equal to 0, then we give him more speed - he goes either to the right, or to the left. It is a throw-like movement.

```c++
        b2Vec2 pos = playerBody->GetPosition();
        view.setCenter(pos.x + screenSize.x / 4, pos.y + screenSize.y / 4);
        window.setView(view);
```

Work with graphics. Get the player position, change the view center and use our view.

```c++
        player.sprite.setPosition(pos.x, pos.y);

        for(int i = 0; i < coin.size(); i++)
            coin[i].sprite.setPosition(coinBody[i]->GetPosition().x, coinBody[i]->GetPosition().y);

        for(int i = 0; i < enemy.size(); i++)
            enemy[i].sprite.setPosition(enemyBody[i]->GetPosition().x, enemyBody[i]->GetPosition().y);
```

Set b2Body positions to the sprites (for the player, coins and enemies).

```c++
        window.clear();

        lvl.Draw(window);

        window.draw(player.sprite);

        for(int i = 0; i < coin.size(); i++)
            window.draw(coin[i].sprite);

        for(int i = 0; i < enemy.size(); i++)
            window.draw(enemy[i].sprite);

        window.display();
```

Clear the window, draw here tiles of the map, then the player, coins and enemies, and then display the window.

```c++
    }

    return 0;
}
```

It's done!

An example map:

![](/assets/img/posts/2013-10-28/header.png)

# Sources

![](/assets/img/posts/2013-10-28/github-logo.png){: .w-25 .normal}

[https://github.com/Izaron/Platformer](https://github.com/Izaron/Platformer)

# Thanks

... to the authors of this [topic](http://en.sfml-dev.org/forums/index.php?topic=3023.0).

... to the authors of Box2D.

... to the authors of [TinyXml](http://www.grinninglizard.com/tinyxml/).
