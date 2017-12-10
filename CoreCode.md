#include <iostream>
#include <fstream>
#include <algorithm>
#include <vector>
#include <string>
#include <cstdlib>
using namespace std;
/**
 * @brief Hlavni soubor programu
 * @file main.cpp
 * @author Jiri Konecny
 * @mainpage KON0327 - Domaci ucetnictvi
 */

 /**
 * @brief Funkce pro ziskani poctu radku
 * @param streamfile   Vstupni soubor, ze ktereho se vypocita pocet radku
 * @return Funkce navraci pocet radku v souboru
 */
int NumberOfLine(ifstream &streamfile)
{
    int LineN = 0;
    string temp;
    while (!streamfile.eof())
    {
        getline(streamfile, temp, '\n');
        LineN++;
    }
    temp.clear();
    return LineN;
}
/** @struct Domacnost
 *  @brief Struktura obsahujici vsechny ucetni data
 *  @param Domacnost.ID            Specificke ID urcene pro identifikaci zaznamu
 *  @param Domacnost.pv            Urceni typu zaznamu (Prijem/Vydej)
 *  @param Domacnost.kategorie     Kategorie zaznamu
 *  @param Domacnost.castka        Castka
 *  @param Domacnost.datum         Datum
 *  @param Domacnost.mesic         Mesic
 */
typedef struct
{
    int ID;
    bool pv;
    string kategorie;
    int castka;
    string datum;
    int mesic;
} Domacnost;
/** @struct Memory_sort
 *  @brief Struktura pracujici jako pomocna pamet pro vypis a serazeni zaznamu
 *  @param Memory_sort.pocetID       Pocet ID zaznamu v mesici
 *  @param Memory_sort.ID_stop       Posledni ID zaznamu v mesici
 *  @param Memory_sort.ID_start      Prvni ID zaznamu v mesici
 */
typedef struct
{
    //int memory[];
    int pocetID;
    int ID_stop;
    int ID_start;
} Memory_sort;

void error(string code);
bool StrToBool(string Type);
string BoolToStr(bool LogicValue);
void Line();
int SumPlus(Domacnost d_data[], int End,int Start);
int SumMinus(Domacnost d_data[], int End,int Start);
int FindMonths(Domacnost d_data[], int Pozition);
bool FillStruct(ifstream &streamfile, Domacnost d_data[]);
bool SetUpSortStruct(Domacnost d_data[], Memory_sort mem_data[], int NumberOfID, int NumberOfMonths);
void SortMaxToMin(Domacnost d_data[],Memory_sort mem_data[], int EndInt, int StartInt,int sump,int sumv);
void SortByCategory(Domacnost d_data[], int NumberOfID, string name, ofstream &html_out);
void WriteSortByCategory(Domacnost d_data[], int NumberOfID);
string NameOfFile();
bool HTML_All(string name, ofstream &html_out, Domacnost d_data[], Memory_sort mem_data[], int suma_p, int suma_v, int startint, int endofint);
bool HTML_Category(string name, string Category, int Number, int CountCat);
bool Menu(Domacnost d_data[], Memory_sort mem_data[], int NumberOfID, int NumberOfMonths);
int SmallMenu();
void WriteSmallMenu();
void WriteMenu();
void CSVChoise();
void MakeNewCSVRule();
string TransMounts(int MoonNumber);
bool CheckData(string data, int pozition);
bool CheckMonth(Domacnost d_data[], int Month, int Pozition, bool Mode);
bool WriteEditCSV(Domacnost d_data[], int NumberOfID,string adress);
bool EditCSV(Domacnost d_data[], int NumberOfID);
bool MakeNewCSV(string adress);

/**
 * @brief Funkce ukoncujici program pri kriticke chybe
 * @param code   Popis chyby
 */
void error(string code="Chyba pri behu programu")
{
    cout << code << endl;
    exit(1);
}
/**
 * @brief Funkce vypisujici radek zapomoci znaku '_'
 */
void Line()
{
    for (int i = 0; i < 55; i++)
    {
        cout << "_";
    }
    cout << endl;
}
/**
 * @brief Funkce vypisujici radek zapomoci znaku '*'
 */
void LineAlt()
{
    for (int i = 0; i < 55; i++)
    {
        cout << "*";
    }
    cout << endl;
}
/**
 * @brief Funkce pro preklad Prijmu/Vydaje do logicke hodnoty
 * @param Type   Prijem/Vydej v textove forme
 */
bool StrToBool(string Type)
{
    if (Type == "Prijem")
        return true;
    else if (Type == "Vydej")
        return false;
    else
        return true;
}
/**
 * @brief Funkce pro preklad Prijmu/Vydaje z logicke hodnoty do textove
 * @param LogicValue   Prijem/Vydej ve tvaru logicke hodnoty
 */
string BoolToStr(bool LogicValue)
{
    if (LogicValue)
        return "Prijem";
    else
        return "Vydej";
}
/**
 * @brief Funkce pro vypocet sumy prijmu
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param End  Posledni zaznam
 * @param Start Prvni zaznam
 */
int SumPlus(Domacnost d_data[], int End,int Start=0)
{
    int sum = 0;
    for (int i = Start; i < End; i++)
    {
        if (d_data[i].pv)
            sum += d_data[i].castka;
    }
    return sum;
}
/**
 * @brief Funkce pro vypocet sumy vydaju
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param End  Posledni zaznam
 * @param Start Prvni zaznam
 */
int SumMinus(Domacnost d_data[], int End,int Start=0)

{
    int sum = 0;
    for (int i = Start; i < End; i++)
    {
        if (!d_data[i].pv)
            sum+= d_data[i].castka;
    }
    return sum;
}
/**
 * @brief Funkce pro nalezeni mesice v zaznamu
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param Pozition Cislo zaznamu
 */
int FindMonths(Domacnost d_data[], int Pozition)
{
    string datum;
    datum += d_data[Pozition].datum[3];
    datum += d_data[Pozition].datum[4];
    return atoi(datum.c_str());
}
/**
 * @brief Funkce pro upravu souboru CSV
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param NumberOfID Pocet zaznamu
 */
bool EditCSV(Domacnost d_data[], int NumberOfID)
{
int IDn,options;
string temp;
char AN;
while (1)
{
 cout <<"Zadete ID zaznamu pro editaci:";
cin>>IDn;
 while (cin.fail()|| IDn > NumberOfID || IDn < 0)
            {
                if (cin.fail())
                {
                    cin.clear();
                    cin.ignore(250,'\n');
                }
            cout <<"Opakujte!\t ID: ";
            cin>>IDn;
            }
Line();
IDn--;
if (IDn==d_data[IDn].ID-1)cout <<"ID nalezeno!: "<<endl;
else {cout <<"ID nenalezeno!"<<endl;return false;}
Line();
cout << "ID\tP/V\tKategorie\tCastka\tDatum" << endl;
cout<<d_data[IDn].ID<<"\t"<<BoolToStr(d_data[IDn].pv)<<"\t"<<d_data[IDn].kategorie<<"\t\t"<<d_data[IDn].castka<<"\t"<<d_data[IDn].datum<<endl;
Line();
while (1)
{
cout <<"1.]Upravit polozku -Prijem/Vydej-:"<<endl;
cout <<"2.]Upravit polozku -Kategorie-:"<<endl;
cout <<"3.]Upravit polozku -Castka-:"<<endl;
cout <<"4.]Upravit polozku -Datum-:"<<endl;
cout <<"Vyberte:"<<endl;
cin>>options;
while (cin.fail() || options > 4 || options < 1)
        {
            if (cin.fail())
            {
                cin.clear();
                cin.ignore(250,'\n');
            }
            cout << "Chybny vstup!\tOpakujte: ";
            cin >> options;
        }
switch (options)
{
             case 1:{
                    cout <<"Prijem/Vydej(1/0): ";
                    cin>>temp;
                    if (temp=="1"){
                    temp=BoolToStr(true);}
                    if(temp=="0"){
                    temp=BoolToStr(false);}
                    while (!CheckData(temp,options))
                    {
                    cout <<"Opakujte!\t Prijem/Vydej(1/0): ";
                    cin>>temp;
                    if (temp=="1"){
                    temp=BoolToStr(true);}
                    if(temp=="0"){
                    temp=BoolToStr(false);}
                    }
                    d_data[IDn].pv=StrToBool(temp);
                    break;
                    }
             case 2:{
                    cout <<"Kategorie: ";cin>>temp;
                    while (!CheckData(temp,options+1))
                    {
                    cout <<"Opakujte!\t Kategorie: ";cin>>temp;
                    }
                    d_data[IDn].kategorie=temp;
                    break;
                    }
             case 3:{
                    cout <<"Castka: ";cin>>temp;
                    while (!CheckData(temp,options+1))
                    {
                    cout <<"Opakujte!\t Castka: ";cin>>temp;
                    }
                    d_data[IDn].castka=atoi(temp.c_str());
                    break;
                    }
             case 4:{
                    cout <<"Datum: ";cin>>temp;
                    while (!CheckData(temp,options+1))
                    {
                    cout <<"Opakujte!\t Datum: ";cin>>temp;
                    }
                    while (!CheckMonth(d_data,FindMonths(d_data,IDn),IDn,true))
                    {
                    cout <<"Opakujte!\t Datum: ";cin>>temp;
                    }
                    d_data[IDn].datum=temp;
                    break;
                    }
            default:return false;
}
cout << "Editovat dalsi polozku zaznamu "<<IDn+1<<" ? [A=ano]: ";
cin>>AN;
Line();
if (cin.fail()||(AN!='A'&&AN!='a'))break;
cout << "ID\tP/V\tKategorie\tCastka\tDatum" << endl;
cout<<d_data[IDn].ID<<"\t"<<BoolToStr(d_data[IDn].pv)<<"\t"<<d_data[IDn].kategorie<<"\t\t"<<d_data[IDn].castka<<"\t"<<d_data[IDn].datum<<endl;
}
cout << "Editovat dalsi zaznam? [A=ano]: ";
cin>>AN;
Line();
if (cin.fail()||(AN!='A'&&AN!='a'))break;
}
cout << "Editace dokoncena!"<<endl;
cout << "Ulozit zmeny? [A=ano]: ";
cin>>AN;
Line();
if (cin.fail()||(AN!='A'&&AN!='a'))return false;
else return true;
}
/**
 *  @brief Funkce pro zapsani upraveneho souboru CSV
 *  @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 *   @param NumberOfID Pocet zaznamu
 *   @param adress Adresa souboru CSV
 */
bool WriteEditCSV(Domacnost d_data[], int NumberOfID,string adress)
{
int pozition=5;
string temp;
ofstream editcsv;
editcsv.open(adress.c_str());
if (!editcsv.is_open())
    {
    error("File cannot be found or read.");
    }
for (int i=0;i<NumberOfID;i++)
 {
  for (int j=1;j<=pozition;j++)
  {
   switch (j)
   {
    case 1:{if (i!=0)editcsv<<"\n";
            editcsv<<d_data[i].ID<<";";break;}
    case 2:{editcsv<<BoolToStr(d_data[i].pv)<<";";break;}
    case 3:{editcsv<<d_data[i].kategorie<<";";break;}
    case 4:{editcsv<<d_data[i].castka<<";";break;}
    case 5:{editcsv<<d_data[i].datum;break;}
   }
  }
 }
return true;
}
/**
 *  @brief Funkce pro kontrolu vstupnich/vzstupnich dat
 *  @param data  I/O Data
 *  @param pozition Pozice dat
 */
bool CheckData(string data, int pozition)
 {
 int TempData=0;
 string TempString="";
switch (pozition)
{
case 1:
     {
     for (int i=0;i<(int)data.size();i++)
    {
    if (data[i]<'0'||data[i]>'9')
    return false;
    }
    if (atoi(data.c_str())<0)
    return false;
    break;
     }
case 2:
    {
    if (data.compare("Prijem")==0||data.compare("Vydej")==0)
    break;
    else return false;
    }
case 3:
    {
    if (data.size()>25)
    return false;
    else break;
    }
case 4:
    {
    for (int i=0;i<(int)data.size();i++)
    {
    if (data[i]<'0'||data[i]>'9')
    return false;
    }
    if (atoi(data.c_str())<0)
    return false;
    break;
    }
case 5:
    {
    if (data.size()<10||data.size()>10)return false;
    if (data[0]<'0'||data[0]>'3')return false;
    if (data[1]<'0'||data[1]>'9')return false;
    if (data[2]!='.'||data[5]!='.')return false;
    if (data[3]<'0'||data[3]>'1')return false;
    if (data[4]<'0'||data[4]>'9')return false;
    TempString+=data[3];
    TempString+=data[4];
    TempData=atoi(TempString.c_str());
    if (TempData>12)return false;
    break;
    }
}
return true;
 }
 /**
 *  @brief Funkce pro kontrolu spravnosti mesicu
 *  @param Domacnost d_data[]  Odkay na strukturu Domacnost
 *  @param Month Cislo mesice
 *  @param Number Cislo/a zaznamu
 *  @param Mode Prepinac modu pouziti
 */
bool CheckMonth(Domacnost d_data[], int Month, int Number, bool Mode)
{
 if (Mode)
 {
if (Number==0)return true;
 for (int i=Number;i>0;i--)
 {
if (d_data[i].mesic<d_data[i-1].mesic)
return false;
 }
 return true;
 }
 else
 {
   for (int i=0;i<Number;i++)
 {
 if (Month==d_data[i].mesic)return true;
 }
 return false;
 }
}
/**
 * @brief Funkce MakeNewCSVRule- vypis moznosti pravidel pri tvoreni noveho CSV souboru
 */
void MakeNewCSVRule()
{
    Line();
    cout << "Pravidla pro tvorbu CSV souboru" << endl;
    Line();
    cout << "\t1.]Soubor musi byt tvoren v mesicni navaznosti(Leden,Unor...)!" << endl;
    cout << "\t2.]Datum se zadava je formatu XX.YY.ZZZZ(29.12.1996!" << endl;
    cout << "\t3.]Program kontroluje spravnost vstupu, presto dbej pokynu!" << endl;
}
 /**
 * @brief Funkce pro vytvoreni noveho souboru CSV
    @param adress Adresa souboru CSV
 */
bool MakeNewCSV(string adress)
{
    MakeNewCSVRule();
    char AN;
    string temp="";
    const int pozition=5;
    int ID=1;
    ofstream NewCsvOut;
    NewCsvOut.open(adress.c_str());
    if (!NewCsvOut.is_open())return false;
    Line();
    while (1)
    {
        for (int i=1; i<=pozition; i++)
        {
            switch (i)
            {
             case 1:{
                 cout <<"ID: "<<ID<<endl;
                 NewCsvOut<<ID<<";";
                 break;
                 }
             case 2:{
                    cout <<"Prijem/Vydej(1/0): ";
                    cin>>temp;
                    if (temp=="1"){
                    temp=BoolToStr(true);}
                    if(temp=="0"){
                    temp=BoolToStr(false);}
                    while (!CheckData(temp,i))
                    {
                    cout <<"Opakujte!\t Prijem/Vydej(1/0): ";
                    cin>>temp;
                    if (temp=="1"){
                    temp=BoolToStr(true);}
                    if(temp=="0"){
                    temp=BoolToStr(false);}
                    }
                    NewCsvOut<<temp+";";
                    break;
                    }
             case 3:{
                    cout <<"Kategorie: ";cin>>temp;
                    while (!CheckData(temp,i))
                    {
                    cout <<"Opakujte!\t Kategorie: ";cin>>temp;
                    }
                    NewCsvOut<<temp+";";
                    break;
                    }
             case 4:{
                    cout <<"Castka: ";cin>>temp;
                    while (!CheckData(temp,i))
                    {
                    cout <<"Opakujte!\t Castka: ";cin>>temp;
                    }
                    NewCsvOut<<temp+";";
                    break;
                    }
             case 5:{
                    cout <<"Datum: ";cin>>temp;
                    while (!CheckData(temp,i))
                    {
                    cout <<"Opakujte!\t Datum: ";cin>>temp;
                    }
                    NewCsvOut<<temp;
                    break;
                    }
            default:return false;
            }
            temp.clear();
        }
        cout << "Pridat dalsi zaznam[A=ano]: ";
        cin>>AN;
        Line();
        if (cin.fail()||(AN!='A'&&AN!='a'))break;
        NewCsvOut<<"\n";
        ID++;
    }
    NewCsvOut.close();
    cout << "CSV soubor byl vytvoren!";
    return true;
}
/**
 * @brief Funkce pro naplneni struktury Domacnost
 * @param streamfile Vstupni soubor
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 */
bool FillStruct(ifstream &streamfile, Domacnost d_data[])
{
    int pozice = 1, i = 0;
    string temp = "";
    streamfile.seekg(ios_base::beg);
    while (!streamfile.eof())
    {
        if (pozice < 5)
            getline(streamfile, temp, ';');
        else
            getline(streamfile, temp, '\n');
        switch (pozice)
        {
        case 1:
        {
            if (!CheckData(temp,pozice))
            error("File load error[type->ID error]");
            if (i!=0)
            {
             for (int j=i-1;j>=0;j--)
             {
             if (atoi(temp.c_str())==d_data[j].ID)
             error("File load error[type->ID error]");
             }
            }
            d_data[i].ID = atoi(temp.c_str());
            break;
        }
        case 2:
        {
            if (!CheckData(temp,pozice))
            error("File load error[type->P/V error]");
            d_data[i].pv = StrToBool(temp);
            break;
        }
        case 3:
        {
            if (!CheckData(temp,pozice))
            error("File load error[type->Kategory error]");
            d_data[i].kategorie = temp;
            break;
        }
        case 4:
        {
            if (!CheckData(temp,pozice))
            error("File load error[type->Castka error]");
            d_data[i].castka = atoi(temp.c_str());
            break;
        }
        case 5:
        {
            if (!CheckData(temp,pozice))
            error("File load error[type->Datum error]");
            d_data[i].datum = temp;
            d_data[i].mesic = FindMonths(d_data, i);
            if (!CheckMonth(d_data,d_data[i].mesic,i,true))
            error("File load error[type->Datum error]");
            break;
        }
        default:
            return false;
        }
        temp.clear();
        if (pozice < 5)
            pozice++;
        else
        {
            pozice = 1;
            i++;
        }
    }
    streamfile.close();
    return true;
}
/**
 * @brief Funkce pro hledani ve strukture Domacnost
 * @param keyword  Hledane slovo
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param NumberOfID Pocet zaznamu
 */
void Find(string keyword,Domacnost d_data[],int NumberOfID)
{
    int n=0;
    for (int i=0; i<NumberOfID; i++)
    {
        if ((d_data[i].datum.find(keyword))!=string::npos)
        {
            cout<<"Nalezena shoda!\t"<<"ID: "<<d_data[i].ID<<"\t ve slove: "<<d_data[i].datum<<endl;
            n++;
            cout << "ID\tP/V\tKategorie\tCastka\tDatum" << endl;
            cout<<d_data[i].ID<<"\t"<<BoolToStr(d_data[i].pv)<<"\t"<<d_data[i].kategorie<<"\t\t"<<d_data[i].castka<<"\t"<<d_data[i].datum<<endl;
            Line();
        }
        if ((d_data[i].kategorie.find(keyword))!=string::npos)
        {
            cout<<"Nalezena shoda!\t"<<"ID: "<<d_data[i].ID<<"\t ve slove: "<<d_data[i].kategorie<<endl;
            n++;
            cout << "ID\tP/V\tKategorie\tCastka\tDatum" << endl;
            cout<<d_data[i].ID<<"\t"<<BoolToStr(d_data[i].pv)<<"\t"<<d_data[i].kategorie<<"\t\t"<<d_data[i].castka<<"\t"<<d_data[i].datum<<endl;
            Line();
        }
        if ((TransMounts(d_data[i].mesic-1).find(keyword))!=string::npos)
        {
            cout<<"Nalezena shoda!\t"<<"ID: "<<d_data[i].ID<<"\t ve slove: "<<TransMounts(d_data[i].mesic-1)<<endl;
            n++;
            cout << "ID\tP/V\tKategorie\tCastka\tDatum" << endl;
            cout<<d_data[i].ID<<"\t"<<BoolToStr(d_data[i].pv)<<"\t"<<d_data[i].kategorie<<"\t\t"<<d_data[i].castka<<"\t"<<d_data[i].datum<<endl;
            Line();
        }
        if ((BoolToStr(d_data[i].pv).find(keyword))!=string::npos)
        {
            cout<<"Nalezena shoda!\t"<<"ID: "<<d_data[i].ID<<"\t ve slove: "<<BoolToStr(d_data[i].pv)<<endl;
            n++;
            cout << "ID\tP/V\tKategorie\tCastka\tDatum" << endl;
            cout<<d_data[i].ID<<"\t"<<BoolToStr(d_data[i].pv)<<"\t"<<d_data[i].kategorie<<"\t\t"<<d_data[i].castka<<"\t"<<d_data[i].datum<<endl;
            Line();
        }
    }
    if (n>0)cout<<"Nalezeno "<<n<<" vysledku."<<endl;
    else cout<<"Zadna shoda!"<<endl;
}

/**
 * @brief Funkce pro trizeni vypisu dle castky(od nejvetsi po nejmensi)
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param Memory_sort mem_data[]  Odkaz na strukturu Memory_sort
 * @param NumberOfMonths Pocet mesicu
 * @param EndInt Konec intervalu vypisu zaznamu
 * @param StartInt Pocatek intervalu vypisu zaznamu
 * @param sump Suma prijmu
 * @param sumv Suma vydaju
 */
void SortMaxToMin(Domacnost d_data[],Memory_sort mem_data[], int EndInt, int StartInt,int sump,int sumv)
{
int NumberOfID=0;
int HelpCount=0;
for (int m=StartInt;m<EndInt;m++)
{
 NumberOfID=mem_data[m].pocetID;
  int *memory=new int[NumberOfID];
     for (int i=mem_data[m].ID_start;i<=mem_data[m].ID_stop;i++)
     {
             memory[HelpCount]=d_data[i].castka;
             HelpCount++;
     }
sort(memory,memory+NumberOfID);
 LineAlt();
 cout<<"Mesic: "<<TransMounts(m)<<endl;
 LineAlt();
 cout << "ID\tP/V\tKategorie\tCastka\tDatum" << endl;
 for (int i=HelpCount;i>=0;i--)
 {
     for (int j=mem_data[m].ID_start;j<=mem_data[m].ID_stop;j++)
     {
    if (d_data[j].castka==memory[i])
    {
            cout << d_data[j].ID;
            cout << "\t" << BoolToStr(d_data[j].pv);
            cout << "\t" << d_data[j].kategorie;
            cout << "\t\t" << d_data[j].castka;
            cout << "\t" << d_data[j].datum << endl;
    }
     }
 }
  HelpCount=0;
  delete [] memory;
}
    Line();
    cout << "\t\t\t\tCelkem prijem: \t" << sump << " ,-Kc" << endl;
    cout << "\t\t\t\tCelkem vydaje: \t" << sumv << " ,-Kc" << endl;
    cout << "\t\t\t\tSuma: \t" << sump - sumv << " ,-Kc" << endl;
    Line();
 }
/**
 * @brief Funkce pro pripravu struktury k pouziti
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param Memory_sort mem_data[]  Odkaz na strukturu Memory_sort
 * @param NumberOfID Pocet zaznamu
 * @param NumberOfMonths Pocet mesicu
 */
bool SetUpSortStruct(Domacnost d_data[], Memory_sort mem_data[], int NumberOfID, int NumberOfMonths)
{
    int n = 0;
    mem_data[0].ID_start = 0;
    for (int j = 0; j < NumberOfMonths; j++)
    {
        if (j != 0)
            mem_data[j].ID_start = (mem_data[j - 1].ID_stop) + 1;
        for (int i = 0; i < NumberOfID; i++)
        {
            if (d_data[i].mesic == j + 1)
            {
                mem_data[j].ID_stop = i;
                n++;
            }
        }
        mem_data[j].pocetID = n;
        n = 0;
    }
    return true;
}
/*
/**
 * @brief Funkce pro vypis cele struktury
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param Memory_sort mem_data[]  Odkaz na strukturu Memory_sort
 * @param endint Konec intervalu vypisu zaznamu
 * @param startint Pocatek intervalu vypisu zaznamu
 * @param suma_p Suma prijmu
 * @param suma_v Suma vydaju
 */
 /*
void WriteSortStruct(Domacnost d_data[], Memory_sort mem_data[], int endint, int startint, int suma_p, int suma_v)
{
    for (int j = startint; j < endint; j++)
    {
        LineAlt();
        cout << "\tMesic: " << TransMounts(j) << endl;
        LineAlt();
        cout << "ID\tP/V\tKategorie\tCastka\tDatum" << endl;
        Line();
        for (int i = mem_data[j].ID_start; i <= mem_data[j].ID_stop; i++)
        {
            cout << d_data[i].ID;
            cout << "\t" << BoolToStr(d_data[i].pv);
            cout << "\t" << d_data[i].kategorie;
            cout << "\t\t" << d_data[i].castka;
            cout << "\t" << d_data[i].datum << endl;
        }
    }
    Line();
    cout << "\t\t\t\tCelkem prijem: \t" << suma_p << " ,-Kc" << endl;
    cout << "\t\t\t\tCelkem vydaje: \t" << suma_v << " ,-Kc" << endl;
    cout << "\t\t\t\tSuma: \t" << suma_p - suma_v << " ,-Kc" << endl;
    Line();
}*/
/**
 * @brief Funkce pro vypis struktury dle kategorii
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param NumberOfID Pocet zaznamu
 */
void WriteSortByCategory(Domacnost d_data[], int NumberOfID)
{
    string temp_cat = "";
    string all_cat = "";
    int number = 0;
    Line();
    cout << "\tKategorie\tCastka[Kc]" << endl;
    Line();
    for (int i = 0; i < NumberOfID; i++)
    {
        temp_cat = d_data[i].kategorie;
        if (all_cat.find(temp_cat) == string::npos)
        {
            all_cat += temp_cat;
            for (int j = 0; j < NumberOfID; j++)
            {
                if (temp_cat == d_data[j].kategorie)
                {
                    if (d_data[j].pv)
                        number += d_data[j].castka;
                    else
                        number -= d_data[j].castka;
                }
            }
            cout <<i+1<<".]\t"<<d_data[i].kategorie << "\t\t" << number << "\t,-Kc" << endl;
            number = 0;
        }
    }
    Line();
    all_cat.erase();
}
/**
 * @brief Funkce pro vypis struktury dle kategorii a vypis do HTML
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param NumberOfID Pocet zaznamu
 */
void SortByCategory(Domacnost d_data[], int NumberOfID)
{
    string name = NameOfFile();
    string temp_cat = "";
    string all_cat = "";
    int number = 0;
    for (int i = 0; i < NumberOfID; i++)
    {
        temp_cat = d_data[i].kategorie;
        if (all_cat.find(temp_cat) == string::npos)
        {
            all_cat += temp_cat;
            for (int j = 0; j < NumberOfID; j++)
            {
                if (temp_cat == d_data[j].kategorie)
                {
                    if (d_data[j].pv)
                        number += d_data[j].castka;
                    else
                        number -= d_data[j].castka;
                }
            }
            if (!HTML_Category(name,d_data[i].kategorie, number,i))
                error("Chyba pri ukladani do HTML!");
            number = 0;
        }
    }
    cout << "Soubor " << name << " byl uspesne ulozen do HTML!" <<" Cesta:..\\vystupnidata\\" + name<<endl;
    all_cat.erase();
}
/**
 * @brief Funkce pro pojmenovani souboru
 */
string NameOfFile()
{
    string name;
    cout << "Zadej nazev souboru :";
    cin >> name;
    if (name.find(".html") == string::npos)
    {
        name += ".html";
        cout << "Autocorrect: pridana pripona .html" << endl;
    }
    return name;
}
/**
 * @brief Funkce pro vypis struktury do HTML
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param Memory_sort mem_data[]  Odkaz na strukturu Memory_sort
 * @param suma_p Suma prijmu
 * @param suma_v Suma vydaju
 * @param endint Konec intervalu vypisu zaznamu
 * @param startint Pocatek intervalu vypisu zaznamu
 */
bool HTML_All(string name,Domacnost d_data[], Memory_sort mem_data[], int suma_p, int suma_v, int startint, int endofint)
{

    ofstream html_out;
    html_out.open(("..\\vystupnidata\\" + name).c_str());
    //html_out.seekg(ios_base::eof);
    if (!html_out.is_open())
        return false;
    html_out << "<html>" << endl;
    html_out << "<head>" << endl;
    html_out << "<meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\">" << endl;
    html_out << "<title>" << name << "</title>" << endl;
    html_out << "</head>" << endl;
    html_out << "<body>" << endl;
    for (int j = startint; j < endofint; j++)
    {
        html_out << "<table width=\"550\">" << endl;
        html_out << "<tr><th width=\"10\">" << TransMounts(j) << "</th>" << endl;
        html_out << "<th width=\"220\">ID</th><th width=\"50\">Prijem/Vydej</th><th width=\"80\">Kategorie</th><th width=\"50\">Castka</th><th width=\"75\">Datum</th>" << endl;
        for (int i = mem_data[j].ID_start; i <= mem_data[j].ID_stop; i++)
        {
            html_out << "<tr><th width=\"220\"></th><td>" << d_data[i].ID << "</td><td>" << BoolToStr(d_data[i].pv) << "</td><td>" << d_data[i].kategorie << "</td><td>" << d_data[i].castka << "</td><td>" << d_data[i].datum << "</td><td>" << endl;
        }
        if (j == endofint - 1)
        {
            html_out << "<tr><th width=\"300\">Prijmu celkem: </th><td>" << suma_p << "</td><th width=\"5\">Kc ,-</th>" << endl;
            html_out << "<tr><th width=\"300\">Vydaje celkem: </th><td>" << suma_v << "</td><th width=\"5\">Kc ,-</th>" << endl;
            html_out << "<tr><th width=\"300\">Suma: </th><td>" << suma_p - suma_v << "</td><th width=\"5\">Kc ,-</th>" << endl;
        }
        html_out << "</table>" << endl;
    }
    html_out << "</body>" << endl;
    html_out << "</html>" << endl;
    html_out.close();
    return true;
}
/**
 * @brief Funkce pro vypis struktury dle kategorii do HTML
 * @param name Jmeno souboru
 * @param Category  Jmeno kategorie
 * @param Number Suma castek kategorie
 * @param CountCat Cislo kategorie
 */
bool HTML_Category(string name,string Category, int Number,int CountCat)
{
    ofstream html_out;
    html_out.open(("..\\vystupnidata\\" + name).c_str(), ios_base::app);
    if (!html_out.is_open())
        return false;
    html_out << "<html>" << endl;
    html_out << "<head>" << endl;
    html_out << "<meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\">" << endl;
    html_out << "<title>" << name << "</title>" << endl;
    html_out << "</head>" << endl;
    html_out << "<body>" << endl;
    html_out << "<table width=\"250\">" << endl;
    if (CountCat==0)html_out << "<th width=\"220\">Kategorie</th><th width=\"50\">Castka</th>" << endl;
    html_out << "</tr><th width=\"220\">"<<CountCat+1<<".)"<< Category << "</th><td>" << Number<< "</td>" << endl;
    html_out << "</table>" << endl;
    html_out << "</body>" << endl;
    html_out << "</html>" << endl;
    html_out.close();
    return true;
}
/**
 * @brief Funkce pro vypis struktury dle kategorii do HTML
 * @param name Jmeno souboru
 * @param keyword  Hledane slovo
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param NumberOfID  Pocet zaznamu
 */
bool HTML_Find(string name,string keyword,Domacnost d_data[],int NumberOfID)
{
     int n=0;
    ofstream html_out;
    html_out.open(("..\\vystupnidata\\" + name).c_str());
    if (!html_out.is_open())
        return false;
    html_out << "<html>" << endl;
    html_out << "<head>" << endl;
    html_out << "<meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\">" << endl;
    html_out << "<title>" << name << "</title>" << endl;
    html_out << "</head>" << endl;
    html_out << "<body>" << endl;
    html_out << "<table width=\"550\">" << endl;
    html_out << "<tr><th width=\"10\">" << keyword << "</th>" << endl;
    for (int i=0; i<NumberOfID; i++)
    {
        if ((d_data[i].datum.find(keyword))!=string::npos)
        {
            html_out<<"<tr><th width=\"300\">Nalezena shoda!\t</th><td>"<<"ID: "<< "</td><td>"<<d_data[i].ID<< "</td>\t ve slove: <td>"<<d_data[i].datum<< "</td><td>" <<endl;
            n++;
            //html_out << "<tr><th width=\"300\">ID\tP/V\tKategorie\t\tCastka\tDatum</th><td>"<<"" << endl;
            html_out<< "<tr><th width=\"220\"></th><td>" <<d_data[i].ID<< "</td>\t<td>"<<BoolToStr(d_data[i].pv)<< "</td>\t<td>"<<d_data[i].kategorie<<"</td>\t<td>"<<d_data[i].castka<< "</td>\t<td>"<<d_data[i].datum<< "</td><td>" <<endl;
            Line();
        }
        if ((d_data[i].kategorie.find(keyword))!=string::npos)
        {
            html_out<<"<tr><th width=\"300\">Nalezena shoda!\t</th><td>"<<"ID: "<<d_data[i].ID<<"\t ve slove: "<<d_data[i].kategorie<< "</td><td>" <<endl;
            n++;
            //html_out<< "<tr><th width=\"300\">ID\tP/V\tKategorie\t\tCastka\tDatum</th><td>"<<"" << endl;
            html_out<< "<tr><th width=\"220\"></th><td>" <<d_data[i].ID<<"</td>\t<td>"<<BoolToStr(d_data[i].pv)<<"</td>\t<td>"<<d_data[i].kategorie<<"</td>\t<td>"<<d_data[i].castka<<"</td>\t<td>"<<d_data[i].datum<< "</td><td>" <<endl;
            Line();
        }
        if ((TransMounts(d_data[i].mesic-1).find(keyword))!=string::npos)
        {
            html_out<<"<tr><th width=\"300\">Nalezena shoda!\t</th><td>"<<"ID: "<<d_data[i].ID<<"\t ve slove: "<<TransMounts(d_data[i].mesic-1)<< "</td><td>" <<endl;
            n++;
            //html_out << "<tr><th width=\"300\">ID\tP/V\tKategorie\t\tCastka\tDatum</th><td>"<<"" << endl;
            html_out<< "<tr><th width=\"220\"></th><td>" <<d_data[i].ID<<"</td>\t<td>"<<BoolToStr(d_data[i].pv)<<"</td>\t<td>"<<d_data[i].kategorie<<"</td>\t<td>"<<d_data[i].castka<<"</td>\t<td>"<<d_data[i].datum<< "</td><td>" <<endl;
            Line();
        }
        if ((BoolToStr(d_data[i].pv).find(keyword))!=string::npos)
        {
            html_out<<"<tr><th width=\"300\">Nalezena shoda!\t</th><td>"<<"ID: "<<d_data[i].ID<<"\t ve slove: "<<BoolToStr(d_data[i].pv)<< "</td><td>" <<endl;
            n++;
            //html_out << "<tr><th width=\"300\">ID\tP/V\tKategorie\t\tCastka\tDatum</th><td>"<<"" << endl;
            html_out<< "<tr><th width=\"220\"></th><td>" <<d_data[i].ID<<"\t"<<BoolToStr(d_data[i].pv)<<"</td>\t<td>"<<d_data[i].kategorie<<"</td>\t<td>"<<d_data[i].castka<<"</td>\t<td>"<<d_data[i].datum<< "</td><td>" <<endl;
            Line();
        }
    }
    if (n>0)html_out<<"<tr><th width=\"300\">Nalezeno "<<n<<" vysledku.</th><td>"<<endl;
    else html_out<<"<tr><th width=\"300\">Zadna shoda!</th><td>"<<endl;
    html_out << "</table>" << endl;
    html_out << "</body>" << endl;
    html_out << "</html>" << endl;
    html_out.close();
    return true;
}
/**
 * @brief Funkce Menu- hlavni menu programu
 * @param Domacnost d_data[]  Odkaz na strukturu Domacnost
 * @param Memory_sort mem_data[]  Odkaz na strukturu Memory_sort
 * @param NumberOfID  Pocet zaznamu
 * @param NumberOfMonths  Pocet mesicu
 * @param adress Adresa souboru
 */
bool Menu(Domacnost d_data[], Memory_sort mem_data[], int NumberOfID, int NumberOfMonths,string adress)
{
    unsigned int options, optionsS = 0, end_int = NumberOfMonths, start_int = 0;
    int suma_p, suma_v;
    bool exit_f = false;
    while (!exit_f)
    {
        WriteMenu();
        cout << "Vyber moznost: ";
        cin >> options;
        while (cin.fail() || options > 7 || options < 1)
        {
            if (cin.fail())
            {
                cin.clear();
                cin.ignore(250,'\n');
            }
            cout << "Chybny vstup!\tOpakujte: ";
            cin >> options;
        }
        switch (options)
        {
        case 1:
        {
            cout << "Pracuji..." << endl;
            end_int = NumberOfMonths;
            start_int = 0;
            suma_p = SumPlus(d_data, NumberOfID);
            suma_v = SumMinus(d_data, NumberOfID);
            SortMaxToMin(d_data,mem_data,end_int, start_int, suma_p, suma_v);
            optionsS = SmallMenu();
            if (optionsS == 1)
            {
                string name = NameOfFile();
                if (!HTML_All(name,d_data, mem_data, suma_p, suma_v, start_int, end_int - 1))
                    error("Chyba pri ukladani do HTML!");
                else
                    cout << "Soubor " << name << " byl uspesne ulozen do HTML!" <<" Cesta:..\\vystupnidata\\" + name<<endl;
                break;
            }
            if (optionsS == 2)
                break;
            if (optionsS == 3 || optionsS == 0)
            {
                exit_f = true;
                break;
            }
        }
        case 2:
        {
            cout << "Pracuji..." << endl;
            WriteSortByCategory(d_data, NumberOfID);
            optionsS = SmallMenu();
            if (optionsS == 1)
            {
                SortByCategory(d_data, NumberOfID);
                break;
            }
            if (optionsS == 2)
                break;
            if (optionsS == 3 || optionsS == 0)
            {
                exit_f = true;
                break;
            }
        }
        case 3:
        {
            cout << "Pracuji..." << endl;
            cout << "Zadejte interval mesicu: (ciselnou hodnotou 1-12!)" << endl;
            cout << "\tOd mesice: ";
            cin >> start_int;
            cout << "\tSlovy: " << TransMounts(start_int-1) << endl;
            cout << "\tPo mesic: ";
            cin >> end_int;
            cout << "\tSlovy: " << TransMounts(end_int-1) << endl;
            while ((cin.fail()) || end_int > 12 || start_int < 1 || end_int < start_int)
            {
                if (cin.fail())
                {
                    cin.clear();
                    cin.ignore(250,'\n');
                }
                cout << "Chybne zadani, opakujte:" << endl;
                cout << "\tOd mesice: ";
                cin >> start_int;
                cout << "\tSlovy: " << TransMounts(start_int-1) << endl;
                cout << "\tDo mesice: ";
                cin >> end_int;
                cout << "\tSlovy: " << TransMounts(end_int - 1) << endl;
            }
            start_int -= 1;
            if((int)end_int>NumberOfMonths)end_int=NumberOfMonths;
            suma_p = SumPlus(d_data, mem_data[end_int-1].ID_stop+1,mem_data[start_int].ID_start);
            suma_v = SumMinus(d_data, mem_data[end_int-1].ID_stop+1,mem_data[start_int].ID_start);
            SortMaxToMin(d_data,mem_data,end_int, start_int, suma_p, suma_v);
            optionsS = SmallMenu();
            if (optionsS == 1)
            {
                string name = NameOfFile();
                if (!HTML_All(name,d_data, mem_data, suma_p, suma_v, start_int, end_int))
                    error("Chyba pri ukladani do HTML!");
                else
                    cout << "Soubor " << name << " byl uspesne ulozen do HTML!" <<" Cesta:..\\vystupnidata\\" + name<<endl;
                break;
            }
            if (optionsS == 2)
                break;
            if (optionsS == 3 || optionsS == 0)
            {
                exit_f = true;
                break;
            }
        }
        case 4:
        {
            cout << "Pracuji..." << endl;
            LineAlt();
            string keyword="";
            cout<<"Zadej hledany vyraz: ";
            cin>>keyword;
            Find(keyword,d_data,NumberOfID);
            optionsS = SmallMenu();
            if (optionsS == 1)
            {
                string name = NameOfFile();
                if (!HTML_Find(name,keyword,d_data,NumberOfID))
                    error("Chyba pri ukladani do HTML!");
                else
                    cout << "Soubor " << name << " byl uspesne ulozen do HTML!" <<" Cesta:..\\vystupnidata\\" + name<<endl;
                break;
            }
            if (optionsS == 2)
                break;
            if (optionsS == 3 || optionsS == 0)
            {
                exit_f = true;
                break;
            }
        }
        case 5:
        {
            cout << "Pracuji..." << endl;
            LineAlt();
            cout <<"Editor souboru CSV. Vstupni soubor:"<<adress<<endl;
            LineAlt();
            if (EditCSV(d_data,NumberOfID))
            {
            if (WriteEditCSV(d_data,NumberOfID,adress))
            cout << "Zmeny byly ulozeny! Adresa souboru:" <<adress<<endl;
            break;
            }
            else
            {
            cout <<"Zmeny nebudou ulozeny!"<<endl;
            cout <<"*Recovery struct Domacnost*"<<endl;
            ifstream tempstremfile;
            tempstremfile.open(adress.c_str());
            if (!tempstremfile.is_open())
                {
            error("File cannot be found or read.");
                }
            if (FillStruct(tempstremfile,d_data))
            {
              cout <<"*Recovery complete!*"<<endl;
              break;
            }
            else error();
            }
        }
        case 6:
            {
            cout << "Pracuji..." << endl;
            return false;
            }
        case 7:
        {
            exit_f = true;
            break;
        }
        default:
        {
            exit_f = true;
            break;
        }
        }
    }
    cout << "Ukoncuji..." << endl;
    return true;
}
/**
 * @brief Funkce pro vypis moznosti v Menu
 */
void WriteMenu()
{
    cout << "MENU:" << endl;
    cout << "\t1.]Vypis prehled vsech mesicu." << endl;
    cout << "\t2.]Vypis prehled vsech kategorii." << endl;
    cout << "\t3.]Vypis dle intervalu." << endl;
    cout << "\t4.]Hledej." << endl;
    cout << "\t5.]Editace CSV souboru!" << endl;
    cout << "\t6.]Reload/Choice/Make CSV" << endl;
    cout << "\t7.]Konec." << endl;
}
/**
 * @brief Funkce pro vypis moznosti v Podmenu
 */
void WriteSmallMenu()
{
    cout << "Volby:" << endl;
    cout << "\t1.]Vypsat do HTML." << endl;
    cout << "\t2.]Navrat do hlavniho menu." << endl;
    cout << "\t3.]Konec programu." << endl;
}
/**
 * @brief Funkce Podmenu- rozsireni hlavniho menu
 */
int SmallMenu()
{
    unsigned int options;
    WriteSmallMenu();
    cout << "Vyber moznost: ";
    cin >> options;
    while (cin.fail() || options > 3 || options < 1)
    {
        if (cin.fail())
        {
            cin.clear();
            cin.ignore(250,'\n');
        }
        cout << "Chybny vstup!\tOpakujte: ";
        cin >> options;
    }
    switch (options)
    {
    case 1:
    {
        cout << "Pracuji..." << endl;
        return 1;
    }
    case 2:
    {
        cout << "Pracuji..." << endl;
        return 2;
    }
    case 3:
    {
        return 3;
    }
    default:
        cout << "Chyba!" << endl;
        return 0;
    }
}
/**
 * @brief Funkce CSVChoise- vzpis moynosti prace s CSV soubory
 */
void CSVChoise()
{
    Line();
    cout << "Vyberte moznost:" << endl;
    cout << "\t1.]Nacist defaultni soubor CSV." << endl;
    cout << "\t2.]Nacist libovolny soubor CSV." << endl;
    cout << "\t3.]Vytvorit novy soubor CSV." << endl;
}
/**
 * @brief Funkce Main- jadro programu, prace se soubory, pouziti inicializacnich funkci
 */
int main()
{
    unsigned int options=0;
    int program_last_id=0;
    int program_last_mesic=0;
    bool Again=true;
    string adress="";
    ifstream read_domacnost_data;
    cout << "Vitejte v programu domaci ucetnictvi!" << endl;
    cout << "Autor: Konecny Jiri, Verze: 1.0 2017" << endl;
    while (Again) {
    CSVChoise();
    cout << "Moznost:";
    cin >> options;
    while (cin.fail() || options > 3 || options < 1)
    {
        if (cin.fail())
        {
            cin.clear();
            cin.ignore(250,'\n');
        }
        cout << "Chybny vstup!\tOpakujte: ";
        cin >> options;
    }
    switch (options)
    {
    case 1:
    {
        adress="..\\vstupnidata\\domacnost_data.csv";
        read_domacnost_data.open(adress.c_str());
        if (!read_domacnost_data.is_open())
        {
            error("File cannot be found or read.");
        }
        cout << "Soubor CSV nastaven, cesta->..\\vstupnidata\\" <<adress<<endl;
        break;
    }
    case 2:
    {
        adress.clear();
        cout << "Zadejte nazev CSV souboru(adresar->..\\vstupnidata\\):";
        cin>>adress;
        if (adress.find(".csv") == string::npos)
        {
            adress += ".csv";
            cout << "Autocorrect: pridana pripona .csv" << endl;
        }
        adress="..\\vstupnidata\\"+adress;
        read_domacnost_data.open(adress.c_str());
        if (!read_domacnost_data.is_open())
        {
            error("File cannot be found or read.");
        }
        cout << "Soubor CSV nastaven, cesta->"<<adress<<endl;
        break;
    }
    case 3:
    {
        adress.clear();
        adress+="..\\vstupnidata\\";
        string name;
        cout << "Zadej nazev CSV souboru :";
        cin >> name;
        if (name.find(".csv") == string::npos)
        {
            name += ".csv";
            cout << "Autocorrect: pridana pripona .csv" << endl;
        }
        adress+=name;
        if (!MakeNewCSV(adress))
        {
            error("Chyba pri tvorbe CSV souboru.");
        }
        read_domacnost_data.open(adress.c_str());
        if (!read_domacnost_data.is_open())
        {
            error("File cannot be found or read.");
        }
        cout << "Soubor CSV nastaven, cesta->"<<adress<<endl;
        break;
    }
    default:
        error();
    }

    program_last_id = NumberOfLine(read_domacnost_data);
    Domacnost *d_data = new Domacnost[program_last_id];
    FillStruct(read_domacnost_data, d_data);

    program_last_mesic = (d_data[program_last_id - 1].mesic);
    Memory_sort *mem_data = new Memory_sort[program_last_mesic];
    SetUpSortStruct(d_data, mem_data, program_last_id, program_last_mesic);
    if (Menu(d_data, mem_data, program_last_id, program_last_mesic,adress))
    {
    Again=false;
    delete[] d_data;
    delete[] mem_data;
    }
    }

    return 0;
}
/**
 * @brief Funkce pro preklad mesicu z cisel na text
 * @param MoonNumber  Ciselna hodnota mesice
 */
string TransMounts(int MoonNumber)
{
    switch (MoonNumber)
    {
    case 0:
    {
        return "Leden";
    }
    case 1:
    {
        return "Unor";
    }
    case 2:
    {
        return "Brezen";
    }
    case 3:
    {
        return "Duben";
    }
    case 4:
    {
        return "Kveten";
    }
    case 5:
    {
        return "Cerven";
    }
    case 6:
    {
        return "Cervenec";
    }
    case 7:
    {
        return "Srpen";
    }
    case 8:
    {
        return "Zari";
    }
    case 9:
    {
        return "Rijen";
    }
    case 10:
    {
        return "Listopad";
    }
    case 11:
    {
        return "Prosinec";
    }
    default:
        return "overfloat";
    }
    return "Chyba!";
}
