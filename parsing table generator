#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define bool char
#define true 1
#define false 0
#define contains equalElem

typedef enum _Type Type;
typedef enum _State State;
typedef enum _MergeMode MergeMode;
typedef enum _ActionType ActionType;
typedef struct _Short Short;
typedef struct _String String;
typedef struct _PrimitiveProduction PrimitiveProduction;
typedef struct _Production Production;
typedef struct _Item Item;
typedef struct _GotoMap GotoMap;
typedef struct _SpreadMap SpreadMap;
typedef struct _Action Action;
typedef struct _List List;
typedef struct _Stack Stack;
typedef struct _Node Node;
typedef List ItemSet;
typedef List ItemSetCollection;

enum _Type
{
	SHORT,
	STRING,
	PRIMITIVE_PRODUCTION,
	PRODUCTION,
	ITEM,
	GOTO_MAP,
	SPREAD_MAP,
	ACTION,
	LIST
};

enum _State
{
	START,
	NONSPACE_TO_SPACE,
	FIND
};

enum _MergetMode
{
	REPEAT_ALLOWED,
	REPEAT_BANNED
};

enum _ActionType
{
	SHIFT_IN,
	REDUCE,
	ACCEPT,
	GOTO
};

struct _Short
{
	Type type;
	unsigned short val;
};

struct _String
{
	Type type;
	char *str;
};

struct _PrimitiveProduction
{
	Type type;
	String *headStr;
	List *bodyStrs;
};

struct _Production
{
	Type type;
	unsigned short head;
	unsigned short *body;
};

struct _Item
{
	Type type;
	unsigned short productionId;
	unsigned short dotPos;
	List *lookAheads;
};

struct _GotoMap
{
	Type type;
	List *itemSet;
	unsigned short symbol;
	List *gotoSet;
};

struct _SpreadMap
{
	Type type;
	Item *spreader;
	Item *spreadee;
};

struct _Action
{
	Type type;
	ActionType actionType;
	unsigned short id;
};

struct _List
{
	Type type;
	Node *head;
	Node *tail;
};

struct _Node
{
	void *elem;
	Node *next;
};

void waitForCommand();
bool initGrammar();
bool readLines();
char *effectiveStr();
bool formPrimitiveProductionSet();
void formOrderedPrimitiveSymbolSet();
void formOrderedPrimitiveSymbolArray();
void formProductions();
ItemSet *CLOSUREForLR0();
ItemSet *CLOSUREForLR1();
ItemSet *GOTOForLR0();
ItemSet *findGOTO();
List *FIRSTForSymbol();
List *FIRSTForSymbolStr();
ItemSetCollection *itemSetCollectionForLR0();
ItemSetCollection *itemSetCollectionForLALR();
Action *generateAction();
void formSelfGeneratedLookAheads();
void formSpreadedLookAheads();
void eliminateLeftRecursion();
void eliminateInstantLeftRecursion();
void eliminateEpsilon();
void kernelize();
bool formParsingTable();
void printParsingTable();
void printProductionTable();
void resume();
void freeParsingTable();
void freeSymbolArray();
void freeProductions();
void printSymbolEnum();
void printEnumStr();
unsigned short *getCatenatedSymbolStr();
unsigned short symbolId();
bool isProduction();
bool isEffectiveStr();
bool isEffectiveHead();
bool hasLeftRecursion();
bool hasEpsilon();
bool spread();
char *strStart();
char *strEnd();
String *strDup();
String *getAuxiliaryStr();
Item *getSpreadee();

Short *newShort();
String *newString();
PrimitiveProduction *newPp();
Item *newItem();
GotoMap *newGotoMap();
SpreadMap *newSpreadMap();
Action *newAction();
List *newList();
void addElem();
void removeElem();
void freeElem();
void freeList();
void printElem();
void printList();
unsigned short listSize();
unsigned short elemId();
bool equals();
void *equalElem();
void addList();
void *clone();

bool containerOnly;
bool listOrderConsidered;
char buffer[65536];
unsigned short productionCount;
unsigned short symbolCount;
unsigned short nonTerminalCount;
unsigned short itemSetCount;
List *lineList;
List *primitiveProductionSet;
List *orderedPrimitiveSymbolSet;
char **orderedPrimitiveSymbolArray;
Production *productions;
List *gotoMapSet;
List *spreadMapSet;
List *lalr;//LALR项集族
Action **parsingTable;
FILE *fout = stdout;
FILE *fin;

void main()
{	
	while (true) {
		waitForCommand();
		if (!initGrammar()) {
			resume();
			continue;
		}
		lalr = itemSetCollectionForLALR();
		if (!formParsingTable()) {
			resume();
			continue;
		}
		printParsingTable();
		printProductionTable();
		resume();
	}
}

/*
接收用户命令以决定是否启动生成语法分析表的工作
*/
void waitForCommand()
{
	char keyin[32];
	
	while (true) {
		printf("按c键启动为桌面的语法文件grammar.txt生成语法分析表的工作：");
		gets(keyin);
		if (!strcmp(keyin, "c") || !strcmp(keyin, "C"))
			break;
		printf("指令错误\n\n");
	}
}

/*
从文件中读取文法，得到以字符串代表每个文法符号的
、有序的文法符号集合（非终结符号在前，终结符号在
后），进而得到以每个文法符号在该集合中的短整型序
号为元素的产生式
*/
bool initGrammar()
{
	if (!readLines())
		return false;
	if (!formPrimitiveProductionSet())
		return false;
	eliminateLeftRecursion();
	formOrderedPrimitiveSymbolSet();
	formOrderedPrimitiveSymbolArray();
	formProductions();
	return true;
}

/*
从文件grammar.txt中读取每行的产生式，存储为一个产生式集合
*/
bool readLines()
{
	unsigned short i = 0;
	char *head, *end, temp;
	char *fileName = "C:\\Users\\lenovo\\Desktop\\grammar.txt";
	String *s;
	fin = fopen(fileName, "r");

	if (!fin) {
		printf("文件grammar.txt打开失败\n\n");
		return false;
	}

	while ((buffer[i++] = fgetc(fin)) != EOF);
	buffer[--i] = '\0';

	head = buffer;

	while (*head) {
		end = strEnd(head, '\n');
		temp = *end;
		*end = '\0';
		s = newString(effectiveStr(head));
		if (!lineList)
			lineList = newList();
		addElem(lineList, s);
		head = temp ? end + 1 : end;
	}

	return true;
}

/*
将字符串str中的由( |\t)+分割的后半部分复制并返回
*/
char *effectiveStr(char *str)
{
	char *end, *end1, *end2, *dup, temp;
	end1 = strEnd(str, ' ');
	end2 = strEnd(str, '\t');
	end = end1 < end2 ? end1 : end2;

	while (*end == ' ' || *end == '\t')
		++end;

	end1 = strEnd(end, '\n');
	temp = *end1;
	*end1 = '\0';
	dup = strdup(end);
	*end1 = temp;

	return dup;
}

/*
基于读取到的产生式集合，把每个产生式中的每个
文法符号都存储为一个对应的字符串
*/
bool formPrimitiveProductionSet()
{
	Node *walk;
	unsigned short i;
	char *line;
	char *start, *end, *headEnd;
	String *newStr, *headStr;
	PrimitiveProduction *pp;
	List *newStrs;
	unsigned short count = 1;

	if (!lineList) {
		printf("文件中无内容\n");
		return false;
	}

	primitiveProductionSet = newList();
	productionCount = 0;
	listOrderConsidered = true;

	for (walk = lineList->head, i = 1; walk; walk = walk->next, ++i) {

		line = ((String*)walk->elem)->str;

		if (!isProduction(line)) {
			printf("第%d行不是一个产生式\n", i);
			return false;
		}

		headEnd = strstr(line, "-->");
		*headEnd = '\0';

		start = strStart(line);
		end = strEnd(start, ' ');
		headStr = strDup(start, end);
		
		newStrs = newList();

		end = headEnd + 3;

		while (*end) {
			start = strStart(end);
			if (!*start)
				break;
			end = strEnd(start, ' ');
			newStr = strDup(start, end);
			addElem(newStrs, newStr);
		}

		if (listSize(newStrs) == 1 &&
			equals(newStrs->head->elem, headStr)) {
			freeElem(newStr);
			freeList(newStrs);
			printf("第%d行不是一个产生式\n", i);
			return false;
		}

		pp = newPp(headStr, NULL);

		if (newStrs->head)
			pp->bodyStrs = newStrs;
		else
			freeList(newStrs);

		if (contains(primitiveProductionSet, pp)) {
			printf("该产生式已经存在\n\n");
			freeElem(pp);
			printf("第%d行的产生式已经存在\n", i);
			return false;
		}

		addElem(primitiveProductionSet, pp);
		productionCount = count;
		++count;
	}

	listOrderConsidered = false;
	return true;
}

/*
消除所输入文法的左递归
*/
void eliminateLeftRecursion()
{
	Node *walk, *walk1, *walk2, *walk3;
	List *nonTerminalStrs = newList();

	for (walk = primitiveProductionSet->head; walk; walk = walk->next) {
		PrimitiveProduction *pp = walk->elem;
		if (!contains(nonTerminalStrs, pp->headStr))
			addElem(nonTerminalStrs, clone(pp->headStr));
	}

	for (walk = nonTerminalStrs->head; walk; walk = walk->next) {
		String *headStr = walk->elem;
		for (walk1 = nonTerminalStrs->head; walk1 != walk; walk1 = walk1->next) {
			String *bodyStr = walk1->elem;
			for (walk2 = primitiveProductionSet->head; walk2;) {
				PrimitiveProduction *pp = walk2->elem;
				walk2 = walk2->next;
				if (pp->bodyStrs && 
					equals(headStr, pp->headStr) &&
					equals(bodyStr, pp->bodyStrs->head->elem)) {
					removeElem(pp->bodyStrs, pp->bodyStrs->head->elem);
					for (walk3 = primitiveProductionSet->head; walk3; walk3 = walk3->next) {
						PrimitiveProduction *pp1 = walk3->elem;
						if (equals(pp1->headStr, bodyStr)) {
							List *list = newList();
							PrimitiveProduction *newpp = newPp(clone(headStr), NULL);
							addList(list, pp1->bodyStrs, REPEAT_ALLOWED);
							addList(list, pp->bodyStrs, REPEAT_ALLOWED);
							if (list->head)
								newpp->bodyStrs = list;
							else
								freeList(list);
							addElem(primitiveProductionSet, newpp);
						}
					}
					removeElem(primitiveProductionSet, pp);
				}
			}
		}
		eliminateInstantLeftRecursion(headStr);
	}

	freeList(nonTerminalStrs);
	productionCount = listSize(primitiveProductionSet);
}

/*
消除在当前文法中以headStr为头的产生式的立即左递归
*/
void eliminateInstantLeftRecursion(String *headStr)
{
	Node *walk;
	char *str;
	String *auxiliary;

	if (!hasLeftRecursion(headStr))
		return;

	str = headStr->str;
	auxiliary = getAuxiliaryStr(str);

	for (walk = primitiveProductionSet->head; walk; walk = walk->next) {
		PrimitiveProduction *pp = walk->elem;
		if (!strcmp(str, pp->headStr->str))
			if (pp->bodyStrs && 
				!strcmp(str, ((String*)pp->bodyStrs->head->elem)->str)) {
				freeElem(pp->headStr);
				pp->headStr = clone(auxiliary);
				removeElem(pp->bodyStrs, pp->bodyStrs->head->elem);
				addElem(pp->bodyStrs, clone(auxiliary));
			} else {
				if (!pp->bodyStrs)
					pp->bodyStrs = newList();
				addElem(pp->bodyStrs, clone(auxiliary));
			}
	}

	addElem(primitiveProductionSet, newPp(auxiliary, NULL));
}

/*
为非终结符号headStr构造辅助非终结符号headStr'，用于左递归的消除
*/
String *getAuxiliaryStr(char *headStr)
{
	size_t len = strlen(headStr);
	String *auxiliary = newString(strdup(headStr));

	auxiliary->str = realloc(auxiliary->str, len + 2);
	auxiliary->str[len] = '1';
	auxiliary->str[len + 1] = '\0';

	return auxiliary;
}

/*
如果在当前文法中有形如headStr-->headStr...的产生式，
返回true，否则返回false
*/
bool hasLeftRecursion(String *headStr)
{
	Node *walk;
	char *str = headStr->str;
	for (walk = primitiveProductionSet->head; walk; walk = walk->next) {
		PrimitiveProduction *pp = walk->elem;
		if (pp->bodyStrs &&
			!strcmp(str, pp->headStr->str) &&
			!strcmp(str, ((String*)pp->bodyStrs->head->elem)->str))
			return true;
	}

	return false;
}

/*
将文法中的每个以字符串表示的文法符
号放入一个文法符号集合中，并且其顺
序为非终结符号在前，终结符号在后，
并加上结束符号'$'
*/
void formOrderedPrimitiveSymbolSet()
{
	Node *walk;
	orderedPrimitiveSymbolSet = newList();
	symbolCount = nonTerminalCount = 0;

	for (walk = primitiveProductionSet->head; walk; walk = walk->next) {
		PrimitiveProduction *pp = walk->elem;
		if (!contains(orderedPrimitiveSymbolSet, pp->headStr)) {
			addElem(orderedPrimitiveSymbolSet, clone(pp->headStr));
			++symbolCount;
			++nonTerminalCount;
		}
	}

	for (walk = primitiveProductionSet->head; walk; walk = walk->next) {
		Node *walk1;
		PrimitiveProduction *pp = walk->elem;
		List *bodyStrs = pp->bodyStrs;
		if (bodyStrs)
			for (walk1 = bodyStrs->head; walk1; walk1 = walk1->next)
				if (!contains(orderedPrimitiveSymbolSet, walk1->elem)) {
					addElem(orderedPrimitiveSymbolSet, clone(walk1->elem));
					++symbolCount;
				}
	}

	addElem(orderedPrimitiveSymbolSet, newString(strdup("$")));
	addElem(orderedPrimitiveSymbolSet, newString(strdup("#")));
	symbolCount += 2;
}

/*
生成链表形式的文法符号集合的数组形式
*/
void formOrderedPrimitiveSymbolArray()
{
	Node *walk;
	unsigned short i, size = listSize(orderedPrimitiveSymbolSet);
	orderedPrimitiveSymbolArray = malloc((size + 1) * sizeof(char*));
	
	for (walk = orderedPrimitiveSymbolSet->head, i = 1; walk; walk = walk->next, ++i) {
		String *str = walk->elem;
		orderedPrimitiveSymbolArray[i] = strdup(str->str);
	}
	
	orderedPrimitiveSymbolArray[0] = "empty string\0";
}

/*
基于用字符串代表文法符号的产生式，
生成用短整型数代表文法符号的产生式
*/
void formProductions()
{
	Node *walk, *walk1;
	unsigned short i, j;
	
	productions = calloc(productionCount, sizeof(Production));

	for (walk = primitiveProductionSet->head, i = 0; walk; walk = walk->next, ++i) {
		PrimitiveProduction *pp = walk->elem;
		productions[i].type = PRODUCTION;
		productions[i].head = symbolId(pp->headStr->str);
		productions[i].body = calloc(listSize(pp->bodyStrs) + 1, sizeof(unsigned short));
		j = 0;
		if (pp->bodyStrs)
			for (walk1 = pp->bodyStrs->head; walk1; walk1 = walk1->next, ++j)
				productions[i].body[j] = symbolId(((String*)walk1->elem)->str);
		productions[i].body[j] = 0;
	}
}

/*
返回字符串文法符号symbolStr的短整型编号
*/
unsigned short symbolId(char *symbolStr)
{
	unsigned short i;
	
	for (i = 1; i <= symbolCount; ++i)
		if (!strcmp(symbolStr, orderedPrimitiveSymbolArray[i]))
			return i;
	
	return 0;
}

/*
判断字符串str是否合法的产生式，
是返回true, 否返回false
*/
bool isProduction(char *str)
{
	char *headEnd = strstr(str, "-->");

	if (!headEnd ||
		headEnd == str) {
		printf("不是产生式\n\n");
		return false;
	}

	*headEnd = '\0';

	if (!isEffectiveStr(str) ||
		!isEffectiveHead(str)) {
		printf("不是产生式\n\n");
		return false;
	}

	*headEnd = '-';

	return true;
}

/*
判断字符串str是否只有一个有效子串
（有效是指，每个字符都不是空格），
是返回true, 否返回false
*/
bool isEffectiveHead(char *str)
{
	unsigned short i;
	State state = START;

	for (i = 1; str[i]; ++i) {
		if (state == START &&
			str[i - 1] != ' ' &&
			str[i] == ' ')
			state = NONSPACE_TO_SPACE;
		if (state == NONSPACE_TO_SPACE &&
			str[i - 1] == ' ' &&
			str[i] != ' ')
			state = FIND;
	}

	return state != FIND;
}

/*
判断字符串str是否包含除空格外的其他字符，
是返回true, 否返回false
*/
bool isEffectiveStr(char *str)
{
	unsigned short i;

	for (i = 0; str[i]; ++i)
		if (str[i] != ' ')
			return true;

	return false;
}

/*
找出字符串str中第一个非空格的字符，
返回该字符的指针（若无非空格的字符，
则返回字符串尾的指针）
*/
char *strStart(char *str)
{
	unsigned short i;

	for (i = 0; str[i]; ++i)
		if (str[i] != ' ')
			break;

	return &str[i];
}

/*
找出字符串str中第一个字符c，
返回该字符的指针（若字符c，
则返回字符串尾的指针）
*/
char *strEnd(char *str, char c)
{
	while (*str && *str != c)
		++str;

	return str;
}

/*
将以start开头，end结尾的内存空间的数据复制
到新的同样大小的空间，将新空间的最后一个
字节置零， 将其打包为一个String并返回
*/
String *strDup(char *start, char *end)
{
	char temp = *end;
	String *dup;

	*end = '\0';
	dup = newString(strdup(start));
	*end = temp;

	return dup;
}

/*
计算出所输入文法的LALR项集族，并返回
*/
ItemSetCollection *itemSetCollectionForLALR()
{
	Node *walk;
	ItemSetCollection *lr0 = itemSetCollectionForLR0();
	
	kernelize(lr0);
	formSelfGeneratedLookAheads(lr0);
	formSpreadedLookAheads(lr0);

	for (walk = lr0->head; walk; walk = walk->next)
		walk->elem = CLOSUREForLR1(walk->elem);

	return lr0;
}

/*
计算出所输入文法的LR0项集族，并返回
*/
ItemSetCollection *itemSetCollectionForLR0()
{
	Node *walk;
	ItemSetCollection *itemSetCollection = newList();
	ItemSet *triggerSet = newList();

	gotoMapSet = newList();

	addElem(triggerSet, newItem(0, 0));
	addElem(itemSetCollection, CLOSUREForLR0(triggerSet));
	++itemSetCount;

	for (walk = itemSetCollection->head; walk; walk = walk->next) {
		ItemSet *itemSet = walk->elem;
		unsigned short symbol;
		for (symbol = 2; symbol < symbolCount - 1; ++symbol) {
			ItemSet *gotoSet = GOTOForLR0(itemSet, symbol);
			if (gotoSet->head) {
				ItemSet *I = equalElem(itemSetCollection, gotoSet);
				GotoMap *map = newGotoMap(itemSet, symbol, NULL);
				if (I) {
					map->gotoSet = I;
					freeList(gotoSet);
				} else {
					map->gotoSet = gotoSet;
					addElem(itemSetCollection, gotoSet);
					++itemSetCount;
				}
				addElem(gotoMapSet, map);
			} else
				freeList(gotoSet);
		}
	}

	return itemSetCollection;
}

/*
计算出项集I的LR0形式的GOTO(I, symbol)函数，并返回
*/
ItemSet *GOTOForLR0(ItemSet *I, unsigned short symbol)
{
	Node *walk;
	ItemSet *gotoSet = newList();

	for (walk = I->head; walk; walk = walk->next) {
		Item *item = walk->elem;
		if (symbol == productions[item->productionId].body[item->dotPos]) {
			Item *copy = clone(item);
			copy->dotPos++;
			addElem(gotoSet, copy);
		}
	}

	return CLOSUREForLR0(gotoSet);
}

/*
计算出项集I的LR0形式的CLOSURE(I)函数，并返回
*/
ItemSet *CLOSUREForLR0(ItemSet *I)
{
	Node *walk;
	
	for (walk = I->head; walk; walk = walk->next) {
		Item *item = walk->elem;
		unsigned short symbol = productions[item->productionId].body[item->dotPos];
		if (symbol > 0 && symbol <= nonTerminalCount) {
			unsigned short i;
			for (i = 0; i < productionCount; ++i)
				if (productions[i].head == symbol) {
					Item *_newItem = newItem(i, 0);
					if (!contains(I, _newItem))
						addElem(I, _newItem);
					else
						freeElem(_newItem);
				}
		}
	}

	return I;
}

/*
删除项集或项集族list中的非内核项
*/
void kernelize(List *list)
{
	switch (*(Type*)list->head->elem) {
		case ITEM:;
		{
			Node *walk;
			for (walk = list->head; walk;) {
				Item *item = walk->elem;
				walk = walk->next;
				if (item->productionId && !item->dotPos)
					removeElem(list, item);
			}
			break;
		}
		case LIST:
		{
			Node *walk;
			for (walk = list->head; walk; walk = walk->next)
				kernelize(walk->elem);
		}
	}
}

/*
为项集族lr0确定它的自发生成的向前看符号
*/
void formSelfGeneratedLookAheads(ItemSetCollection *lr0)
{
	Node *walk, *walk1, *walk2, *walk3;
	Short *symbolAux = newShort(symbolCount);
	Short *symbolEnd = newShort(symbolCount - 1);//symbolId("$") = symbolCount - 1
	ItemSet *headSet;
	Item *headItem;
	
	spreadMapSet = newList();

	for (walk = lr0->head; walk; walk = walk->next) {
		ItemSet *I = walk->elem;
		for (walk1 = I->head; walk1; walk1 = walk1->next) {
			ItemSet *J = newList();
			Item *kernel = walk1->elem;
			Item *copy = clone(kernel);
			copy->lookAheads = newList();
			addElem(copy->lookAheads, clone(symbolAux));
			addElem(J, copy);
			J = CLOSUREForLR1(J);
			for (walk2 = J->head; walk2; walk2 = walk2->next) {
				Item *item = walk2->elem;
				unsigned short symbol = productions[item->productionId].body[item->dotPos];
				if (symbol) {
					Item *spreadee;
					ItemSet *gotoSet = findGOTO(I, symbol);
					item->dotPos++;
					spreadee = equalElem(gotoSet, item);
					if (!spreadee->lookAheads)
						spreadee->lookAheads = newList(); 
					for (walk3 = item->lookAheads->head; walk3; walk3 = walk3->next) {
						Short *sh = walk3->elem;
						if (sh->val == symbolCount)
							addElem(spreadMapSet, newSpreadMap(kernel, spreadee));
						else if (!contains(spreadee->lookAheads, sh))
							addElem(spreadee->lookAheads, clone(sh));
					}
				}
			}
			freeList(J);
		}
	}

	headSet = lr0->head->elem;
	headItem = headSet->head->elem;
	headItem->lookAheads = newList();
	addElem(headItem->lookAheads, symbolEnd);
}

/*
将向前看符号在项集族lr0中传播，直到没有符号被传播为止
*/
void formSpreadedLookAheads(ItemSetCollection *lr0)
{
	Node *walk, *walk1;
	bool spreadContinue;

	while (true) {
		spreadContinue = false;
		for (walk = lr0->head; walk; walk = walk->next) {
			ItemSet *I = walk->elem;
			for (walk1 = I->head; walk1; walk1 = walk1->next)
				if (spread(walk1->elem))
					spreadContinue = true;
		}
		if (!spreadContinue)
			break;
	}
}

/*
计算出项集I的LR1形式的CLOSURE(I)函数，并返回
*/
ItemSet *CLOSUREForLR1(ItemSet *I)
{
	Node *walk, *walk1;
	unsigned short i;

	for (walk = I->head; walk; walk = walk->next) {
		Item *item = walk->elem;
		unsigned short symbol = productions[item->productionId].body[item->dotPos];
		if (symbol > 0 && symbol <= nonTerminalCount) {
			List *lookAheads = item->lookAheads;
			for (walk1 = lookAheads->head; walk1; walk1 = walk1->next) {
				Short *sh = walk1->elem;
				unsigned short *betaAStr = 
					getCatenatedSymbolStr(
					productions[item->productionId].body + item->dotPos + 1,
					sh->val);
				List *first = FIRSTForSymbolStr(betaAStr);
				free(betaAStr);
				for (i = 0; i < productionCount; ++i) {
					if (productions[i].head == symbol) {
						Item *item, *_newItem = newItem(i, 0);
						item = equalElem(I, _newItem);
						if (item) {
							addList(item->lookAheads, first, REPEAT_BANNED);
							freeElem(newItem);
						} else {
							List *list = newList();
							addList(list, first, REPEAT_ALLOWED);
							_newItem->lookAheads = list;
							addElem(I, _newItem);
						}
					}
				}
				freeList(first);
			}
		}
	}

	return I;
}

/*
从gotoMapSet中找出项集I的GOTO(I, symbol)，并返回，
若未找到则返回NULL
*/
ItemSet *findGOTO(ItemSet *I, unsigned short symbol)
{
	Node *walk;

	for (walk = gotoMapSet->head; walk; walk = walk->next) {
		GotoMap *map = walk->elem;
		if (map->itemSet == I && map->symbol == symbol)
			return map->gotoSet;
	}

	return NULL;
}

/*
将项item的向前看符号传播给它能传播到的项，
若有新符号被传播，则返回true，否则返回false
*/
bool spread(Item *spreader)
{
	Node *walk;
	bool spreadDetected = false;

	for (walk = spreadMapSet->head; walk; walk = walk->next) {
		SpreadMap *map = walk->elem;
		if (spreader == map->spreader) {
			unsigned short prevLen = listSize(map->spreadee->lookAheads);
			addList(map->spreadee->lookAheads, spreader->lookAheads, REPEAT_BANNED);
			if (prevLen != listSize(map->spreadee->lookAheads))
				spreadDetected = true;
		}
	}

	return spreadDetected;
}

/*
计算出文法符号symbol的FIRST函数，并返回
*/
List *FIRSTForSymbol(unsigned short symbol)
{
	unsigned short i;
	List *list = newList();

	if (symbol > symbolCount ||
		symbol == 0)
		return list;

	if (symbol > nonTerminalCount) {
		addElem(list, newShort(symbol));
		return list;
	}

	for (i = 0; i < productionCount; ++i)
		if (productions[i].head == symbol) {
			if (!productions[i].body)
				addElem(list, newShort(0));
			else {
				List *list1 = FIRSTForSymbolStr(productions[i].body);
				addList(list, list1, REPEAT_BANNED);
				freeList(list1);
			}
		}

	return list;
}

/*
计算出文法符号串symbolStr的FIRST函数，并返回
*/
List *FIRSTForSymbolStr(unsigned short *symbolStr)
{
	unsigned short i;
	List *list = newList();

	for (i = 0; symbolStr[i]; ++i) {
		List *list1 = FIRSTForSymbol(symbolStr[i]);
		if (!list1->head)
			return list;
		if (!hasEpsilon(list1)) {
			addList(list, list1, REPEAT_BANNED);
			freeList(list1);
			return list;
		}
		eliminateEpsilon(list1);
		addList(list, list1, REPEAT_BANNED);
		freeList(list1);
	}

	addElem(list, newShort(0));

	return list;
}

/*
判断一个文法符号集中是否有epsilon
*/
bool hasEpsilon(List *list)
{
	Node *walk;

	for (walk = list->head; walk; walk = walk->next)
		if (*(Type*)walk->elem == SHORT &&
			!((Short*)walk->elem)->val)
			return true;

	return false;
}

/*
删除一个文法符号集中的epsilon
*/
void eliminateEpsilon(List *list)
{
	Node *walk;

	if (!list)
		return;

	for (walk = list->head; walk;) {
		void *elem = walk->elem;
		walk = walk->next;
		if (*(Type*)elem == SHORT && !((Short*)elem)->val) {
			removeElem(list, elem);
			break;
		}
	}
}

/*
将以start为起始地址的文法符号串
与向前看符号lookAhead拼接成新的
文法符号串，并返回
*/
unsigned short *getCatenatedSymbolStr(unsigned short *start,
									  unsigned short lookAhead)
{
	unsigned short *symbolStr;
	unsigned short i = 0;
	
	while (start[i])
		++i;
	
	symbolStr = malloc((i + 2) * sizeof(unsigned short));
	for (i = 0; start[i]; ++i)
		symbolStr[i] = start[i];
	
	symbolStr[i] = lookAhead;
	symbolStr[i + 1] = 0;

	return symbolStr;
}

/*
根据项集族lalr生成语法分析表，
成功返回true，失败返回false
*/
bool formParsingTable()
{
	Node *walk, *walk1;
	unsigned short symbol;
	parsingTable = calloc((symbolCount - 2) * itemSetCount, sizeof(Action*));

	for (walk = lalr->head; walk; walk = walk->next) {
		ItemSet *I = walk->elem;
		for (walk1 = I->head; walk1; walk1 = walk1->next) {
			Item *item = walk1->elem;
			unsigned short symbol = productions[item->productionId].body[item->dotPos];
			if (symbol > nonTerminalCount && symbol < symbolCount - 1 || !symbol) {
				unsigned short base = (elemId(lalr, I) - 1) * (symbolCount - 2);
				if (symbol) {
					unsigned short index = base + symbol - 2;
					unsigned short id = elemId(lalr, findGOTO(I, symbol)) - 1;
					if (!(parsingTable[index] =
						generateAction(parsingTable[index], newAction(SHIFT_IN, id))))
						return false;
				} else {
					if (!item->productionId) {
						Short *sh = newShort(symbolCount - 1);
						if (contains(item->lookAheads, sh)) {
							unsigned short index = base + symbolCount - 3;//symbolCount - 2 = symbolId("$") - 1
							if (!(parsingTable[index] =
								generateAction(parsingTable[index], newAction(ACCEPT, 0))))
								return false;
						}
						free(sh);
					} else {
						Node *walk;
						for (walk = item->lookAheads->head; walk; walk = walk->next) {
							Short *sh = walk->elem;
							unsigned short index = base + sh->val - 2;
							if (!(parsingTable[index] =
								generateAction(parsingTable[index], newAction(REDUCE, item->productionId))))
								return false;
						}
					}
				}
			}
		}
	}

	for (walk = lalr->head; walk; walk = walk->next) {
		ItemSet *I = walk->elem;
		for (symbol = 2; symbol <= nonTerminalCount; ++symbol) {
			ItemSet *gotoSet = findGOTO(I, symbol);
			if (gotoSet) {
				unsigned short base = (elemId(lalr, I) - 1) * (symbolCount - 2);
				unsigned short index = base + symbol - 2;
				unsigned short stateId = elemId(lalr, gotoSet) - 1;
				if (!(parsingTable[index] =
					generateAction(parsingTable[index], newAction(GOTO, stateId))))
					return false;
			}
		}
	}

	printf("\n语法分析表已生成\n");
	return true;
}

/*
决定动作action是否可以填充到语法分析表中的某个位置，
若之前该位置为空，则返回action；若之前该位置不为空，
且其上的动作与action相同，则释放掉action，返回之前
的动作；若其上的动作与action不同，则产生一个语法分
析表的冲突，说明该文法不是LALR的，返回NULL
*/
Action *generateAction(Action *prev, Action *action)
{
	if (!prev)
		return action;
	else if (equals(prev, action)) {
		free(action);
		return prev;
	} else {
		printf("该文法不是LALR的\n");
		return NULL;
	}
}

/*
打印语法分析表
*/
void printParsingTable()
{
	char keyin[64];
	bool forParser = false;
	unsigned short i, j;
	Node *walk;

	printf("\n\n将要打印语法分析表，请选择打印格式\n");

	while (true) {
		printf("按1选择默认格式，按2选择语法分析器源码的初始化数据格式：");
		gets(keyin);
		if (!strcmp(keyin, "1") || !strcmp(keyin, "2"))
			break;
		printf("指令错误\n\n");
	}

	if (!strcmp(keyin, "2"))
		forParser = true;

	printf("\n请选择打印位置\n");

	while (true) {
		printf("按1选择在本窗口打印，按2选择在桌面生成文件grammar parsing table.txt并在其中打印：");
		gets(keyin);
		if (!strcmp(keyin, "1") || !strcmp(keyin, "2"))
			break;
		printf("指令错误\n\n");
	}

	if (!strcmp(keyin, "2")) {
		char *fileName = "C:\\Users\\lenovo\\Desktop\\grammar parsing table.txt";
		fout = fopen(fileName, "w");
		if (!fout) {
			printf("文件创建失败\n");
			exit(1);
		}
	}

	printSymbolEnum();

	if (forParser) {
		fprintf(fout, "const Action parsingTable[%d][%d] = \n{ ", itemSetCount, symbolCount - 2);
		for (i = 0; i < itemSetCount; ++i) {
			if (i > 0)
				fprintf(fout, ",\n  ");
			fprintf(fout, "{");
			for (j = 2; j < symbolCount; ++j) {
				unsigned short base = i * (symbolCount - 2);
				Action *action = parsingTable[base + j - 2];
				if (j > 2)
					fprintf(fout, ", ");
				fprintf(fout, "{");
				if (action) {
					switch (action->actionType) {
						case SHIFT_IN:
							fprintf(fout, "SHIFT_IN, %d", action->id);
							break;
						case REDUCE:
							fprintf(fout, "REDUCE, %d", action->id - 1);
							break;
						case GOTO:
							fprintf(fout, "GOTO, %d", action->id);
							break;
						case ACCEPT:
							fprintf(fout, "ACCEPT, ");
					}
				} else
					fprintf(fout, "NONE, ");
				fprintf(fout, "}");
			}
			fprintf(fout, "}");
		}
		fprintf(fout, " };\n\n");
		fprintf(fout, "const Production productions[%d] = \n{", productionCount - 1);
		for (walk = primitiveProductionSet->head->next, i = 0; walk; walk = walk->next, ++i) {
			PrimitiveProduction *pp = walk->elem;
			if (i)
				fprintf(fout, ", ");
			if (i && !(i % 5))
				fprintf(fout, "\n");
			if (i)
				fprintf(fout, " ");
			fprintf(fout, "{");
			printEnumStr(pp->headStr->str);
			fprintf(fout, ", %d}", listSize(pp->bodyStrs));
		}
		fprintf(fout, "};\n\n");
	} else
		for (i = 0; i < itemSetCount; ++i)
			for (j = 2; j < symbolCount; ++j) {
				unsigned short base = i * (symbolCount - 2);
				if (parsingTable[base + j - 2]) {
					fprintf(fout, "Action[%d, %s] = ", i, orderedPrimitiveSymbolArray[j]);
					printElem(parsingTable[base + j - 2]);
					fprintf(fout, "\n");
				}
			}

	if (fout != stdout) {
		for (i = 0; i < 8; ++i)
			fprintf(fout, "\n");
		fclose(fout);
	}

	printf("\n语法分析表打印成功\n\n");
}

/*
打印经过左递归消除的产生式列表
*/
void printProductionTable()
{
	Node *walk;
	unsigned short i;
	char *fileName = "C:\\Users\\lenovo\\Desktop\\productions table.txt";
	
	fout = fopen(fileName, "w");

	if (!fout) {
		printf("经左递归消除的产生式文件生成失败\n");
		return;
	}

	for (walk = primitiveProductionSet->head->next, i = 1; walk; walk = walk->next, ++i) {
		fprintf(fout, "%d ", i - 1);
		printElem(walk->elem);
		fprintf(fout, "\n");
	}

	fclose(fout);

	printf("经左递归消除的产生式文件productions table.txt已在桌面生成\n\n\n");
}

/*
打印出所有的文法符号的C语言枚举形式
*/
void printSymbolEnum()
{
	unsigned short i;

	fprintf(fout, "enum _GrammarSymbol\n{\n");
	
	for (i = 2; i <= nonTerminalCount; ++i) {
		fprintf(fout, "\t");
		printEnumStr(orderedPrimitiveSymbolArray[i]);
		fprintf(fout, ",\n");
	}

	for (i = nonTerminalCount + 1; i < symbolCount; ++i) {
		fprintf(fout, "\t");
		printEnumStr(orderedPrimitiveSymbolArray[i]);
		if (i < symbolCount - 1)
			fprintf(fout, ",");
		fprintf(fout, "\n");
	}

	fprintf(fout, "};\n\n");
}

/*
将字符串str以枚举类型常量的大写形式打印
*/
void printEnumStr(char *str)
{
	while (*str) {
		if (*str >= 'a' && *str <= 'z')
			fprintf(fout, "%c", *str - 32);
		else if (*str >= 'A' && *str <= 'Z')
			fprintf(fout, "_%c", *str);
		else
			fprintf(fout, "%c", *str);
		++str;
	}
}

/*
释放文法符号字符串数组的空间
*/
void freeSymbolArray()
{
	unsigned short i;

	if (!orderedPrimitiveSymbolArray)
		return;

	for (i = 1; i <= symbolCount; ++i)
		free(orderedPrimitiveSymbolArray[i]);

	free(orderedPrimitiveSymbolArray);
}

/*
释放产生式的空间
*/
void freeProductions()
{
	unsigned short i;
	
	if (!productions)
		return;

	for (i = 0; i < productionCount; ++i)
		free(productions[i].body);
	
	free(productions);
}

/*
释放语法分析表的空间
*/
void freeParsingTable()
{
	unsigned short i;

	if (!parsingTable)
		return;

	for (i = 0; i < (symbolCount - 2) * itemSetCount; ++i)
		if (parsingTable[i])
			free(parsingTable[i]);

	free(parsingTable);
}

/*
释放堆区，并把所有数据恢复到开始时的状态
*/
void resume()
{
	freeList(lineList);
	freeList(primitiveProductionSet);
	freeList(orderedPrimitiveSymbolSet);
	freeList(gotoMapSet);
	freeList(spreadMapSet);
	freeSymbolArray();
	freeProductions();
	freeParsingTable();
	
	lineList = NULL;
	primitiveProductionSet = NULL;
	orderedPrimitiveSymbolSet = NULL;
	gotoMapSet = NULL;
	spreadMapSet = NULL;
	orderedPrimitiveSymbolArray = NULL;
	productions = NULL;
	parsingTable = NULL;

	containerOnly = listOrderConsidered = false;
	productionCount = 0;
	symbolCount = 0;
	nonTerminalCount = 0;
	itemSetCount = 0;

	fout = stdout;
	fin = NULL;
}

/*
以val为值构造一个新的Short结构体
*/
Short *newShort(unsigned short val)
{
	Short *sh = calloc(1, sizeof(Short));
	sh->type = SHORT;
	sh->val = val;
	return sh;
}

/*
以str为值构造一个新的String结构体
*/
String *newString(char *str)
{
	String *s = calloc(1, sizeof(String));
	s->type = STRING;
	s->str = str;
	return s;
}

/*
以headStr和bodyStrs为值构造一个新的PrimitiveProduction结构体
*/
PrimitiveProduction *newPp(String *headStr, List *bodyStrs)
{
	PrimitiveProduction *pp = calloc(1, sizeof(PrimitiveProduction));
	pp->type = PRIMITIVE_PRODUCTION;
	pp->headStr = headStr;
	pp->bodyStrs = bodyStrs;
	return pp;
}

/*
以id和pos为值构造一个新的Item结构体
*/
Item *newItem(unsigned short id, unsigned short pos)
{
	Item *item = calloc(1, sizeof(Item));
	item->type = ITEM;
	item->productionId = id;
	item->dotPos = pos;
	return item;
}

/*
以itemSet, symbol和gotoSet为值构造一个新的GotoMap结构体
*/
GotoMap *newGotoMap(List *itemSet, unsigned short symbol, List *gotoSet)
{
	GotoMap *map = calloc(1, sizeof(GotoMap));
	map->type = GOTO_MAP;
	map->itemSet = itemSet;
	map->symbol = symbol;
	map->gotoSet = gotoSet;
	return map;
}

/*
以spreader和spreadee为值构造一个新的SpreadMap结构体
*/
SpreadMap *newSpreadMap(Item *spreader, Item *spreadee)
{
	SpreadMap *map = calloc(1, sizeof(SpreadMap));
	map->type = SPREAD_MAP;
	map->spreader = spreader;
	map->spreadee = spreadee;
	return map;
}

/*
以actionType和id为值构造一个新的Action结构体
*/
Action *newAction(ActionType actionType, unsigned short id)
{
	Action *action = calloc(1, sizeof(Action));
	action->type = ACTION;
	action->actionType = actionType;
	action->id = id;
	return action;
}

/*
构造一个新的以elem为唯一元素的List结构体
*/
List *newList()
{
	List *list = calloc(1, sizeof(List));
	list->type = LIST;
	return list;
}

/*
规定了判断两个实例是否相等的法则
*/
bool equals(void *elem1, void *elem2)
{
	Type type1, type2;

	if (elem1 == elem2)
		return true;

	if (!elem1 || !elem2)
		return false;

	type1 = *(Type*)elem1;
	type2 = *(Type*)elem2;

	if (type1 != type2)
		return false;

	switch (type1) {
		case SHORT:
		{
			Short *sh1 = elem1, *sh2 = elem2;
			return sh1->val == sh2->val;
		}
		case STRING:
		{
			String *str1 = elem1, *str2 = elem2;
			return !strcmp(str1->str, str2->str);
		}
		case ITEM:
		{
			Item *item1 = elem1, *item2 = elem2;
			return item1->dotPos == item2->dotPos &&
				item1->productionId == item2->productionId;
		}
		case PRIMITIVE_PRODUCTION:
		{
			PrimitiveProduction *pp1 = elem1, *pp2 = elem2;
			return equals(pp1->headStr, pp2->headStr)
				&& equals(pp1->bodyStrs, pp2->bodyStrs);
		}
		case ACTION:
		{
			Action *a1 = elem1, *a2 = elem2;
			return a1->actionType == a2->actionType
				&& a1->id == a2->id;
		}
		case LIST:
		{
			unsigned short size1 = listSize(elem1);
			unsigned short size2 = listSize(elem2);
			
			List *list1 = elem1, *list2 = elem2;
			Node *walk1, *walk2;

			if (size1 != size2)
				return false;
		
			if (listOrderConsidered) {
				for (walk1 = list1->head, walk2 = list2->head;
					walk1 && walk2; walk1 = walk1->next, walk2 = walk2->next)
					if (!equals(walk1->elem, walk2->elem))
						break;
				return walk1 == walk2;
			} else {
				//此处判断2个List是否相等（不考虑元素的排列顺序，只要含有相同的元素即相等，此算法对包含重复元素的list无效）
				for (walk1 = list1->head; walk1; walk1 = walk1->next) {
					bool flag = false;
					for (walk2 = list2->head; walk2; walk2 = walk2->next)
						if (equals(walk1->elem, walk2->elem)) {
							flag = true;
							break;
						}
					if (!flag)
						return false;
				}
				return true;
			}
		}
		default:
			return elem1 == elem2;
	}
}

/*
返回list中第1个与elem相等的元素
*/
void *equalElem(List *list, void *elem)
{
	Node *walk;
	
	if (!list)
		return NULL;

	for (walk = list->head; walk; walk = walk->next)
		if (equals(walk->elem, elem))
			return walk->elem;
	
	return NULL;
}

/*
返回元素elem在list中的序号
*/
unsigned short elemId(List *list, void *elem)
{
	Node *walk;
	unsigned short i;

	for (walk = list->head, i = 1; walk; walk = walk->next, ++i)
		if (equals(elem, walk->elem))
			return i;

	return 0;
}

/*
将集合list2中的元素并入集合list1
*/
void addList(List *list1, List *list2, MergeMode mode)
{
	Node *walk;

	if (!list1 || !list2)
		return;
	
	for (walk = list2->head; walk; walk = walk->next)
		if (mode == REPEAT_ALLOWED || !contains(list1, walk->elem))
			addElem(list1, clone(walk->elem));
}

/*
克隆元素elem，并返回其克隆体
*/
void *clone(void *elem)
{
	if (!elem)
		return NULL;

	switch (*(Type*)elem) {
		case SHORT:
		{
			Short *sh = elem;
			return newShort(sh->val);
		}
		case STRING:
		{
			String *s = elem;
			return newString(strdup(s->str));
		}
		case PRIMITIVE_PRODUCTION:
		{
			PrimitiveProduction *pp = elem;
			return newPp(clone(pp->headStr), clone(pp->bodyStrs));
		}
		case ITEM:
		{
			Item *item = elem;
			Item *copy = newItem(item->productionId, item->dotPos);
			if (item->lookAheads)
				copy->lookAheads = clone(item->lookAheads);
			return copy;
		}
		case LIST:
		{
			List *list = elem;
			List *copy = newList();
			addList(copy, list, REPEAT_ALLOWED);
			return copy;
		}
		default:
			return NULL;
	}
}

/*
将元素elem添加进集合list
*/
void addElem(List *list, void *elem)
{
	Node *newNode = NULL;
	
	if (!list || !elem)
		return;

	newNode = calloc(1, sizeof(Node));
	newNode->elem = elem;

	if (!list->head)
		list->head = list->tail = newNode;
	else {
		list->tail->next = newNode;
		list->tail = newNode;
	}
}

/*
将元素elem从集合list中删除
*/
void removeElem(List *list, void *elem)
{
	Node **walk = NULL, *prev;

	if (!list)
		return;

	for (prev = NULL, walk = &list->head; *walk;)
		if ((*walk)->elem == elem) {
			Node *node = *walk;
			*walk = node->next;
			if (!containerOnly)
				freeElem(node->elem);
			else if (*(Type*)node->elem == LIST)
				freeList(node->elem);
			free(node);
			if (node == list->tail)
				list->tail = prev;
			break;
		} else {
			prev = *walk;
			walk = &(*walk)->next;
		}
}

/*
打印元素elem
*/
void printElem(void *elem)
{
	if (!elem)
		return;

	switch (*(Type*)elem) {
		case SHORT:
			fprintf(fout, "%s ", orderedPrimitiveSymbolArray[((Short*)elem)->val]);
			break;
		case STRING:
			fprintf(fout, "%s ", ((String*)elem)->str);
			break;
		case PRIMITIVE_PRODUCTION:
		{
			PrimitiveProduction *pp = elem;
			printElem(pp->headStr);
			fprintf(fout, "--> ");
			printElem(pp->bodyStrs);
			break;
		}
		case PRODUCTION:
		{
			Production *p = elem;
			unsigned short i;

			fprintf(fout, "%s-->", orderedPrimitiveSymbolArray[p->head]);
			for (i = 0; p->body[i]; ++i)
				fprintf(fout, "%s ", orderedPrimitiveSymbolArray[p->body[i]]);
			break;
		}
		case ITEM:
		{
			unsigned short i;
			Item *item = elem;
			fprintf(fout, "%s-->", orderedPrimitiveSymbolArray[productions[item->productionId].head]);
			for (i = 0; i < item->dotPos; ++i)
				fprintf(fout, "%s ", orderedPrimitiveSymbolArray[productions[item->productionId].body[i]]);
			fprintf(fout, ". ");
			while (productions[item->productionId].body[i])
				fprintf(fout, "%s ", orderedPrimitiveSymbolArray[productions[item->productionId].body[i++]]);
			if (item->lookAheads) {
				fprintf(fout, "\t");
				printElem(item->lookAheads);
			}
			break;
		}
		case SPREAD_MAP:
		{
			SpreadMap *map = elem;
			fprintf(fout, "item ");
			printElem(map->spreader);
			fprintf(fout, "spreads to item ");
			printElem(map->spreadee);
			break;
		}
		case ACTION:
		{
			Action *action = elem;
			switch (action->actionType) {
				case SHIFT_IN:
					fprintf(fout, "移入状态%d", action->id);
					break;
				case REDUCE:
					fprintf(fout, "按产生式%d：", action->id - 1);
					printElem(&productions[action->id]);
					fprintf(fout, "归约");
					break;
				case ACCEPT:
					fprintf(fout, "ACCEPT");
					break;
				case GOTO:
					fprintf(fout, "goto状态%d", action->id);
			}
			break;
		}
		case LIST:
			printList(elem);
	}
}

/*
打印集合list
*/
void printList(List *list)
{
	Node *walk = NULL;

	if (!list)
		return;

	for (walk = list->head; walk; walk = walk->next) {
		Type type = *(Type*)walk->elem;
		printElem(walk->elem);
		if (type != SHORT && type != STRING)
			fprintf(fout, "\n");
	}
}

/*
释放元素elem及其相关的内存空间
*/
void freeElem(void *elem)
{
	if (!elem)
		return;

	switch (*(Type*)elem) {
		case SHORT:
		case GOTO_MAP:
		case SPREAD_MAP:
		case ACTION:
			free(elem);
			break;
		case STRING:
			free(((String*)elem)->str);
			free(elem);
			break;
		case PRIMITIVE_PRODUCTION:
		{
			PrimitiveProduction *pp = elem;
			freeElem(pp->headStr);
			freeList(pp->bodyStrs);
			free(pp);
			break;
		}
		case ITEM:
		{
			Item *item = elem;
			if (item->lookAheads)
				freeList(item->lookAheads);
			free(elem);
			break;
		}
		case LIST:
			freeList(elem);
	}
}

/*
释放集合list，若containerOnly
为false，则其中的元素也要释放
*/
void freeList(List *list)
{
	if (!list)
		return;

	while (list->head) {
		Node *node = list->head;
		list->head = node->next;
		if (!containerOnly)
			freeElem(node->elem);
		else if (*(Type*)node->elem == LIST)
			freeList(node->elem);
		free(node);
	}

	free(list);
}

/*
返回集合list中的元素的数量
*/
unsigned short listSize(List *list)
{
	unsigned short size = 0;
	Node *node;

	if (!list)
		return 0;

	node = list->head;

	while (node) {
		++size;
		node = node->next;
	}

	return size;
}








