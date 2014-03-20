# How to use cocos2d-x3.0 making a sliding picture game: The first part


Program Screenshot:

This is a complete picture :

![][p1]

This is an upset in the image:

![][p2]

In this new tutorial, we will conquer a new game --- sliding picture games. You definitely know what kind of game , the player's task is to disrupt the first picture , then put a good fight to disrupt the picture . ( Of course , after the end of the tutorial , gentlemen, you can improve the picture upset by the program , and then let the player direct reduction picture , best to get hold of the complete picture for reference ah ! ^ ^ ) Make this type of game , the biggest benefit The next is that we can learn and understand Tiled Map game pave the way for us.

So , in order to make such a game , what we need to do things? Here are the steps to making a list of slide pictures of the game :

1. Create a "Tile" class that contains sprite, position (x, y) and the value of these instance variables.
2. Create a management class , which is responsible for creating all of Tile, also can track the status of all of the Tile .
3. Add touch components, so that the player can swap two tile positions.
4. add some code to randomly load images , so that the game can have more tricks.

So much that when I write out the steps when it is not that easy ? ( Translator: add, see my tutorial friend , not just limited to the specific technical details , to think more creative step game , this game has four steps , your short game needs to think more about it , so ? do not look at the tutorial , it did not start the tutorial ) next, I will step by step to achieve all of the features you will find it is really very simple.

Really, this simple game example , give you a lot of inspiration. . . In the future, based on this series I will be tutorials to introduce you to more of this type of game ( Translator: such as puzzles you, Huarong friends, Sokoban friends, lianliankan friends )

Anyway , let us realize this game to say ! To accomplish this daunting task , we need some auxiliary classes. These classes are Tile.h, Tile.cpp, Box.h, Box.cpp.

If you read this site other tutorials , you must have known "SceneManager" and "PlayLayer" category , and there is no longer a long-winded . If you have not , please refer to [《cocos2d菜单教程》][1]和[《cocos2d精灵教程》][2] .

First, let us look to achieve Tile class.

Tile.h：

<pre>
```cpp
#include "cocos2d.h"
#include "Constants.h"

class TileElem :
	public cocos2d::Object
{
public:
	TileElem();
	~TileElem();

	bool initWithPos(int posX, int posY);
	static TileElem* create(int posX, int posY);
	bool nearTile(TileElem *otherTile);
	void trade(TileElem *otherTile);
	cocos2d::Point pixPosition();

	CC_SYNTHESIZE_READONLY(int, _x, PosX);
	CC_SYNTHESIZE_READONLY(int, _y, PosY);
	CC_SYNTHESIZE_RETAIN(cocos2d::Sprite*, _sprite, Sprite);
	CC_SYNTHESIZE(int, _value, Value);
};
```
</pre>

tile principal is to represent extracted from a big picture of a piece of content out there , there are X, Y position ( note that here x, y coordinates are not equal to the wizard's location coordinates, the position coordinates of the wizard is sprite-> getPosition () ) , elves , have value. Where value can be any value , for example ( this is the case , we can use these locations to determine whether the player makes up a complete map of the right ) can represent the position of each tile in the original image , in this version, we do not uses this value;

Now, let us look at the specific implementation:

Tile.cpp:

<pre>
```cpp
#include "Tile.h"
USING_NS_CC;

TileElem::TileElem()
{
}

TileElem::~TileElem()
{
    CC_SAFE_RELEASE_NULL(_sprite);
}

TileElem* TileElem::create(int posX, int posY)
{
	TileElem *pRet = new TileElem();
	if (pRet && pRet->initWithPos(posX, posY))
	{
		pRet->autorelease();
		return pRet;
	}
	else
	{
		delete pRet;
		pRet = NULL;
		return NULL;
	}
}

bool TileElem::initWithPos(int posX, int posY)
{
	_x = posX;
	_y = posY;
    
    _value = 0;

    _sprite = Sprite::create();
    CC_SAFE_RETAIN(_sprite);
    
	return true;
}

bool TileElem::nearTile(TileElem *otherTile)
{
	return (_x == otherTile->getPosX() &&
		abs(_y - otherTile->getPosY()) == 1) ||
		(_y == otherTile->getPosY() &&
		abs(_x - otherTile->getPosX()) == 1);
}

void TileElem::trade(TileElem *otherTile)
{
	Sprite *tempSprite = Sprite::createWithTexture(_sprite->getTexture());
	tempSprite->setPosition(_sprite->getPosition());
	int tempValue = _value;
	this->setSprite(otherTile->getSprite());
	this->setValue(otherTile->getValue());
	otherTile->setSprite(tempSprite);
	otherTile->setValue(tempValue);
}

Point TileElem::pixPosition()
{
	return Point(kStartX + _x * kTileSize + kTileSize / 2.0f, kStartY + _y * kTileSize + kTileSize / 2.0f);
}
```
</pre>
Most of the content will be able to see to understand - we realized the four methods "initWithPos", "nearTile", "trade" and "pixPosition".

"InitWithPos" approach can be seen from the name what it is doing --- it is class initialization code Tile , Tile it is initialized when the class is called . It receives an x, y values ​​, and the values ​​of these two independent world coordinates , but with the relevant class Box containing them . For example , we use initWithPos (3,4), if we are , then the box 7 * 7 , then this Tile is placed at 3,4 position ) we can use pixPosition function to calculate each Tile Wizard screen position.

"NearTile" receives an argument of type Tile , Tile to determine whether the two are neighbors, if it is , it returns true, otherwise it returns false.

"Trade" is to swap the two Tile wizard . Swap two variables , I believe will be learned C language , the definition of a temporary variable temp, then temp = a; a = b; b = temp;

Finally , "pixPosition" calculated for each Tile Wizard correct coordinates of the location on the screen --- you will see special purpose of this function later.

   The main function is to handle all the Box class to create a single Tile class , loads the appropriate wizard , and place them in the correct position on the screen .

Box.h:

<pre>
```cpp
#include "Tile.h"
USING_NS_CC;

class Box :
	public Object
{
public:
	Box();
	~Box();

	bool initWithSize(Size aSize, int aImgValue);
	TileElem* objectAtPos(int posX, int posY);
	bool check();
	static Box* create(Size size, int factor);

	CC_SYNTHESIZE_READONLY(Size, _size, Size);
	CC_SYNTHESIZE(Layer*, _layer, Layer);
	CC_SYNTHESIZE(bool, lock, Lock);
public:
    std::vector<Vector<TileElem *>> contentVec;
	Vector<TileElem *> readyToRemoveTilesVec;
	
	TileElem* OutBorderTile;
	int imgValue;
};
```
</pre>

Single spectacle file , you can guess some of the content it 's a role. . . size is the size that we are going to create a grid ( 3 * 3, 4 * 5 , 5 * 3,7 * 7 , and so on )

The main Box class two variables is "contentVec" and "readyToRemoveTilesVec". Here comes cocos2d explain there is a Vector, but the editors at the time did not exist found cocos2d for Vector Vector, so I used the C + + standard std :: vector to store . Realization of two-dimensional array .

contentVec variable is actually a multi-dimensional array , at least one dimension ( then if SIZE is 1 * 1 ) . We will create a std :: vector <Vector <TileElem *>>, then each column and then add a Vector <TileElem *> a row. We can use the "return contentVec.at (posY) at (posX);." To get the correct Tile.

readyToRemove variable , in this tutorial, just initialized , but in the future, I will introduce another game , I'll be in there a lot of use of this variable ---- In this tutorial, I will use it to load all the newly created wizard.

Next, let us look at the Box class implementation :

Box.cpp:

<pre>
```cpp
#include "Box.h"

Box::Box()
{
}

Box::~Box()
{
}

Box* Box::create(Size aSize, int aImgValue)
{
	Box *pRet = new Box();
	if (pRet && pRet->initWithSize(aSize, aImgValue))
	{
		pRet->autorelease();
		return pRet;
	}
	else
	{
		delete pRet;
		pRet = NULL;
		return NULL;
	}
}

bool Box::initWithSize(Size aSize, int aImgValue)
{
	imgValue = aImgValue;
	_size = aSize;
	OutBorderTile = TileElem::create(-1, -1);

	for (int y = 0; y < _size.height; y++) {

		Vector<TileElem *> rowContentVec;
		for (int x = 0; x < _size.width; x++) {
			TileElem *tile = TileElem::create(x, y);
			rowContentVec.pushBack(tile);
			readyToRemoveTilesVec.pushBack(tile);
		}
		contentVec.push_back(rowContentVec);
	}
	return true;
}

TileElem* Box::objectAtPos(int posX, int posY)
{
	if (posX < 0 || posX >= kBoxWidth || posY < 0 || posY >= kBoxHeight) {
		return OutBorderTile;
	}

	return contentVec.at(posY).at(posX);
}

bool Box::check()
{
	int countTile = readyToRemoveTilesVec.size();
	if (0 == countTile){
		return false;
	}

	for (int i = 0; i < countTile; i++) {
		TileElem *tile = readyToRemoveTilesVec.at(i);
		tile->setValue(0);
		if (tile->getSprite()) {
			_layer->removeChild(tile->getSprite());
		}
	}
	readyToRemoveTilesVec.clear();

	char name[20] = { 0 };
	sprintf(name, "%d.png", imgValue);
	Texture2D * texture = Director::getInstance()->getTextureCache()->addImage(name);

	Vector<SpriteFrame *> imgFrames;

	for (int i = 0; i < 7; i++) {
		for (int j = 6; j >= 0; j--) {
			SpriteFrame *imgFrame = SpriteFrame::createWithTexture(texture, Rect(i * 40, j * 40, 40, 40)); 
			imgFrames.pushBack(imgFrame);
		}
	}
	for (int x = 0; x < _size.width; x++) {
		int extension = 0;
		for (int y = 0; y < _size.height; y++) {
			TileElem *tile = TileElem::create(x, y);
			if (tile->getValue() == 0){
				extension++;
			} else if(extension == 0) {
			}
		}
        
		for (int i = 0; i < extension; i++) {
			TileElem *destTile = this->objectAtPos(x, kBoxHeight - extension + i);
			SpriteFrame * img = imgFrames.at(0);
			Sprite *sprite = Sprite::createWithSpriteFrame(img);
			imgFrames.eraseObject(img);
			sprite->setPosition(kStartX + x * kTileSize + kTileSize / 2, kStartY + (kBoxHeight + i) * kTileSize + kTileSize / 2 - kTileSize * extension);
			_layer->addChild(sprite);
			destTile->setValue(imgValue);
			destTile->setSprite(sprite);
		}
	}

	return true;
}

```
</pre>

So, in the end of this class done thing --- I think "initWithSize" and "objectAtPos" These two methods have been very clear. . There is no longer a long-winded .

Therefore, the most important methods --- check. First, we determine if it contains any readyToRemoveTilesVec Tiles. . In our example, all the Tile are inside. . . Very good ! Then we traverse the array of all the elements inside , put them one by one all the elves removed , and finally, we put all the elements in the entire array empty . Now, you may know, when making a game of multiple levels of when and how to do clean-up of the work.

Next, we load the textures ( such as 1.png, 2.png, 3.png , etc. ) from a resource file, and then store them in a Texture2D object , after which we will build all objects from Texture2D sprite frames. We will create 49 Elf frames. Because our image size is 280 * 280 , so each CCSpriteFrame size is 40 * 40. There used a double for the cycle .

<pre>
```cpp
for (int x = 0; x < _size.width; x++) {
	int extension = 0;
	for (int y = 0; y < _size.height; y++) {
		TileElem *tile = TileElem::create(x, y);
		if (tile->getValue() == 0){
			extension++;
		} else if(extension == 0) {
		}
	}
```
</pre>
 

This part of the code above , for this tutorial is actually not necessary, but it can be used to determine how many need to be replaced Tile picture . This mechanism is very good, especially when you make a Tile drop games ( such as Bejeweled ) time, because once you do not want to replace all of the Tile. . . In this example , we will detect Tile 's value attribute is 0 , then use the extension variable to track how many value of 0 Tile. In this case , since all the readyToRemoveTilesVec Tile exist , so extension variable is always 7.

Here , we get a first Y coordinate (kBoxHeight-extension + i) of Tile. . Because, in our case , kBoxHeight = 7, and the extension is always 7 , so we really only need to worry about the variable i on it. i would have been incremented from 0 to 6 . Okay, you might ask , why do I do it? Because, in fact, I would use the game to now if we are familiar with the case, future work will be very easy :) . Next, look at how the elves are working it. . .

<pre>
```cpp
for (int i = 0; i < extension; i++) {
	TileElem *destTile = this->objectAtPos(x, kBoxHeight - extension + i);
	SpriteFrame * img = imgFrames.at(0);
	Sprite *sprite = Sprite::createWithSpriteFrame(img);
	imgFrames.eraseObject(img);
	sprite->setPosition(kStartX + x * kTileSize + kTileSize / 2, kStartY + (kBoxHeight + i) * kTileSize + kTileSize / 2 - kTileSize * extension);
	_layer->addChild(sprite);
	destTile->setValue(imgValue);
	destTile->setSprite(sprite);
}
```
</pre>

Then , we create a new Sprite, through a given SpriteFrame, and the wizard box placed in the right position . We put all the imgValue are set to Tile same .

Finally , take a look PlayLayer class, we should be very familiar. . . It will handle touches, as well as initialization box class. We need to track the two previous touch - "selectedTile" variable refers to the current generation of players selected Tile, this time, if the player chooses when another Tile , Tile and in front of it will be exchanged. Next, look at the concrete realization of it.

PlayLayer.h:

<pre>
```cpp
#include "cocos2d.h"
#include "Box.h"
#include "Constants.h"
USING_NS_CC;

class PlayLayer :
	public Layer
{
public:
	PlayLayer();
	~PlayLayer();

	bool init();
	void check(Object* pSender, void* data);
	void changeWithTileA(TileElem* a, TileElem* b);
	void onTouchesBegan(const std::vector<Touch*>& touches, Event *unused_event);

	CREATE_FUNC(PlayLayer);

public:
	Box* box;
	TileElem* selectedTile;
	int value;
};
```
</pre>

Next is its implementation :

PlayLayer.cpp:

<pre>
```cpp
#include "PlayLayer.h"

PlayLayer::PlayLayer()
{
}

PlayLayer::~PlayLayer()
{
    CC_SAFE_RELEASE_NULL(box);
}

bool PlayLayer::init()
{
	if (!Layer::init())
	{
		return false;
	}

	value = CCRANDOM_0_1()*kKindCount + 1;
    box = Box::create(Size(kBoxWidth, kBoxHeight), value);
    CC_SAFE_RETAIN(box);
	box->setLayer(this);
	box->setLock(true);
	box->check();

	auto listener = EventListenerTouchAllAtOnce::create();
	listener->onTouchesBegan = CC_CALLBACK_2(PlayLayer::onTouchesBegan, this);
	this->_eventDispatcher->addEventListenerWithSceneGraphPriority(listener, this);

	selectedTile = NULL;

	return true;
}

void PlayLayer::onTouchesBegan(const std::vector<Touch*>& touches, Event *unused_event)
{
    CCLOG("touched");
    
	auto touch = touches.at(0);
	auto location = touch->getLocation();

	if (location.y < kStartY ||
		location.y >(kStartY + (kTileSize * kBoxHeight)) ||
		location.x < kStartX ||
		location.x >(kStartX + (kTileSize * kBoxWidth)))
	{
		return;
	}

	int x = (location.x - kStartX) / kTileSize;
	int y = (location.y - kStartY) / kTileSize;

	if (selectedTile && selectedTile->getPosX() == x && selectedTile->getPosY() == y) {
		selectedTile = NULL;
		return;
	}

	TileElem *tile = box->objectAtPos(x, y);
	if (tile->getPosX() >= 0 && tile->getPosY() >= 0) {
		if (selectedTile && selectedTile->nearTile(tile)) {
			box->setLock(true);
			this->changeWithTileA(selectedTile, tile);
			
			selectedTile = NULL;
		}
		else {
			if (selectedTile) {
				if (selectedTile->getPosX() == x && selectedTile->getPosY() == y) {
					selectedTile = NULL;
				}
			}
			selectedTile = tile;
		}
	}
}

void PlayLayer::check(Object* pSender, void* data)
{
    CCLOG("PlayLayer::check has been called");
}

void PlayLayer::changeWithTileA(TileElem* a, TileElem* b)
{
    CCLOG("PlayLayer::changeWithTileA has been called");
    Action *actionA = Sequence::create(
		MoveTo::create(kMoveTileTime, b->pixPosition()),
		CallFuncN::create(CC_CALLBACK_1(PlayLayer::check, this, (void*)a)),
		NULL);

	Action *actionB = Sequence::create(
		MoveTo::create(kMoveTileTime, a->pixPosition()),
		CallFuncN::create(CC_CALLBACK_1(PlayLayer::check, this, (void*)b)),
		NULL);

	a->getSprite()->runAction(actionA);
	b->getSprite()->runAction(actionB);

	a->trade(b);
}
```
</pre>

This class there are three methods --- init method is used to initialize the box class , onTouchesBegan method to determine which Tile is user selectable, if the newly selected Tile Tile then equal to the original direct return ; If the neighboring Tile , then it exchange ; otherwise , do not do anything. When switching Tile , call the Sequence and MoveTo to show animated exchange. In fact , the code we really exchanged only a-> trade (b).

This chapter [源码][3]

I hope you enjoyed this tutorial .

 See next tutorial !

[p1]: ./res/course_screenshot1.jpg " tutorial screenshot"
[p2]: ./res/course_screenshot2.jpg " tutorial screenshot"


[1]: wating "cocos2d-x menu tutorial"
[2]: wating "cocos2d-x wizard tutorial"
[3]: ./SlideImageGame.zip " source"
