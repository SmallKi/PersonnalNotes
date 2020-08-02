# Window

1. sf::Window对象

   1. 构造方式

      ```c++
      sf Window::Window()   //默认构造，并不真正的构造，要使用create()方法创造
      
      sf Window::Window(VideoMode mode,    //显示模式
                         const String& title,   //标题
                         Unideal style = Style::Default  //窗口的样式
                         const ContextSettings & settings = ContextSettings() //opengl设置？
                        )
      ```

      ps: VideoMode可以通过sf::VideoMode::getDesktopMode()来获取

   2. 成员方法

      ```c++
      bool isOpen();  //是否打开状态
      void close();  //关闭窗口
      Vector2 getPosition(); //获取位置信息
      void setPosition(Vector2 pos);  //设置窗口位置
      //Vector2<T>(T x, T y)  定义了一个二维的容器
      void setTilte(const string& title)  //设置标题
      void sf::Window::setFramerateLimit(unsigned int limit) //设置最大帧率	
      
      ```

      

# Texture

纹理类（几乎所有的GPU都会对纹理的大小有所限制，可以通过静态方法getMaximumSize()来获得）

这个类是将图片放进显卡里，以便于快速绘制

1. 构造方法

   ```c++
   Texture();   //empty
   Texture(const Texture &copy);  //copy constructor    
   ```

2. 成员方法

   ```c++
   bool create(unsigned int width, unsigned int height); //失败的话没有改变
   bool loadFromFile(const std::string &filename, const IntRect &area=IntRect());  //从文件加载图片
   //IntRect 是 Rect<int>的别名
   
   bool loadFromImage(const Image& img);  //直接从Image图像中加载

   bool setSmooth(bool tag);    //是否开启平滑（对于原像素大小的，不需要开，那些需要放大缩小之类的，可以考虑开。）
   
   Image copyToImage(void);   //转为图像对象，耗费很大，慎用 
   ```
   
   

# Image

图像类

用于操作图像，渲染图像要用texture

1. 构造方法

   默认构造即可

2. 成员方法

   ```c++
   //创建图像
   bool create(unsigned int width,  //宽度
               unsigned int height,  //高度
               const Color &color=Color(0, 0, 0)//颜色
              );
   
   //从图片创建
   bool loadFromFile(const std::string &filename);
   
   //保存图片
   bool saveToFile (const std::string &filename) const;
   
   //获取图片尺寸    
   Vector2u getSize() const;
   
   //水平翻转图片
   void flipHorizontally();
   
   //垂直翻转图片
   void flipVertically();
   
   //获取像素的颜色
   Color getPixcil(unsigned int x, unsigned int y);  //不会检测x或y的合法性，超过会发生未定义行为
   
   //设置像素颜色
   void setPixcel(unsigned int x, unsigned int y, Color color); 
   ```





# Color

定义颜色

1. 构造方法

   ```c++
   Color(Uint8 red, Uint8 green, Uint8 blue, Uint8 alpha=255);   //Uint8指代的是unsigned char类型
   
   Color();  //默认构造
   
   COlor(Uint32 color);
   
   //PUBLIC MEMBER
   sf::Color black       = sf::Color::Black;
   sf::Color white       = sf::Color::White;
   sf::Color red         = sf::Color::Red;
   sf::Color green       = sf::Color::Green;
   sf::Color blue        = sf::Color::Blue;
   sf::Color yellow      = sf::Color::Yellow;
   sf::Color magenta     = sf::Color::Magenta;
   sf::Color cyan        = sf::Color::Cyan;
   sf::Color transparent = sf::Color::Transparent;
   ```

2. 成员方法

   ```c++
   Uint32 toInteger() const;  //获取颜色
   ```




# Shape类

Shape 类继承自sf::Drawable 和 sf::Transformed

这意味着shape的实例可以进行绘制和移动

Shape类中可以叠加纹理，但注意，传递的是纹理的指针而不是引用！！

```c++
//当给Shape添加纹理时，默认会拉伸图像以适应形状
//如果想要纹理能够重复填充，参考下面的代码

	sf::Texture tex;   //创建纹理
	tex.loadFromFile("test.jpg",sf::IntRect(0,0,50,50));  //加载纹理
	tex.setRepeated(true);   //使得纹理可以重复
	sf::RectangleShape rect(sf::Vector2f(200.f, 200.f));  //创建依赖的形状

	rect.setTextureRect(sf::IntRect(0,0,200,200));  //必须设置一个比单个纹理纹理更大的区域来承载图像
	rect.setTexture(&tex);   //传指针
```



1. 常用方法

   ```c++
    //返回性状的尺寸，w,h
   Vector2 getSize(); 
   
   //设置位置
   void setPosition(Vector2f pos);  //设置的点是形状的原点，默认是左上角
   
   //改变形状的原点
   void setOrigion(Vector2f ori);
   
   ```

   

# Sprite精灵类

可以将精灵类理解为简单的Shape类，然后附加了一个Texture纹理

Sprite的大小是由纹理决定的，如果要显示设置大小，需要setTextureRect去承载



1. 构造方法

   ```c++
   //默认构造
   Sprite();
   
   Sprite(const Texture &tex);  //创建有纹理的精灵
   
   Sprite(const Texture &texture, const IntRect &rectangle);   //进行裁剪纹理的精灵
   ```

2. 常用成员方法

   ```c++
   void setTexture(const &Texture tex);    //给精灵设置纹理
   
   void setTextureRect(const &IntRect rect);  //给精灵的纹理裁剪
   
   const Texture& getTexture();   //获取纹理对象
   
   
   FloatRect getLocalBounds()	const;   //获取图像的局部边界
   //height = 32 width=32 left = 100 top=100
   //保存了origin的坐标信息，和原始宽高（不包括移动、旋转和scale）
   
   FloatRect getGlobalBounds()	const;   //获取图像的全局边界，经过转换后的
   ```




# SoundBuffer类

存储音效，与Sound类的关系相当于texture和sprite的关系



![1580388914336](C:\Users\12440\AppData\Roaming\Typora\typora-user-images\1580388914336.png)

SoundBuffer可以同时多个播放

Music只能同一时间播放一个

# Music类

与SoundBuffer类的关系就像是shape和sprite的关系

music适合长音乐



# 常用方法

```c++
void sf::Sleep(unsigned int);   //休眠多少秒
```

