#include <iostream>
#include <vector>
#include <fstream>
#include <string>

using namespace std;

void Login();
void ShowMainMenue();
void GoBackToMainMenu();
void ShowQuickWithdrawScreen();
void ShowNormalWithdrawScreen();

struct sClient
{
    string AccountNumber;
    string PinCode;
    string Name;
    string Phone;
    double AccountBalance;
    bool MarkForDelete = false;
};

const string ClientFileName = "Clients.txt";

sClient CurrentClient;

vector<string> SplitString(string S1, string Delim)
{
    vector<string> vString;
    short pos = 0;
    string sWord; // define a string variable  

    // use find() function to get the position of the delimiters  
    while ((pos = S1.find(Delim)) != std::string::npos)
    {
        sWord = S1.substr(0, pos); // store the word   
        if (sWord != "")
        {
            vString.push_back(sWord);
        }

        S1.erase(0, pos + Delim.length());  /* erase() until positon and move to next word. */
    }

    if (S1 != "")
    {
        vString.push_back(S1); // it adds last word of the string.
    }

    return vString;

}

sClient ConvertLinetoRecord(string Line, string Seperator = "#//#")
{
    sClient Client;
    vector<string> vClientData;
    vClientData = SplitString(Line, Seperator);

    Client.AccountNumber = vClientData[0];
    Client.PinCode = vClientData[1];
    Client.Name = vClientData[2];
    Client.Phone = vClientData[3];
    Client.AccountBalance = stod(vClientData[4]);//cast string to double
    return Client;
}

string ConvertRecordToLine(sClient Client, string Seperator = "#//#")
{
    string stClientRecord = "";
    stClientRecord += Client.AccountNumber + Seperator;
    stClientRecord += Client.PinCode + Seperator;
    stClientRecord += Client.Name + Seperator;
    stClientRecord += Client.Phone + Seperator;
    stClientRecord += to_string(Client.AccountBalance);
    return stClientRecord;
}

vector <sClient> SaveCleintsDataToFile(string FileName, vector <sClient> vClients)
{
    fstream MyFile;
    MyFile.open(FileName, ios::out);//overwrite

    string DataLine;

    if (MyFile.is_open())
    {
        for (sClient C : vClients)
        {

            if (C.MarkForDelete == false)
            {
                //we only write records that are not marked for delete.  
                DataLine = ConvertRecordToLine(C);
                MyFile << DataLine << endl;
            }

        }

        MyFile.close();
    }

    return vClients;
}

vector <sClient> LoadCleintsDataFromFile(string FileName)
{
    vector <sClient> vClients;
    fstream MyFile;
    MyFile.open(FileName, ios::in);//read Mode

    if (MyFile.is_open())
    {
        string Line;
        sClient Client;

        while (getline(MyFile, Line))
        {
            Client = ConvertLinetoRecord(Line);
            vClients.push_back(Client);
        }
        MyFile.close();
    }
    return vClients;
}

bool DepositBalancyToClientByAccountNumber(string AccountNumber, double Amount, vector <sClient>& vClients)
{
    char Answer = 'n';
    cout << "\nAre you sure you want to preform this transaction ? y/n ";
    cin >> Answer;

    if (toupper(Answer) == 'Y')
    {
        for (sClient& Client : vClients)
        {
            if (Client.AccountNumber == AccountNumber)
            {
                Client.AccountBalance += Amount;
                SaveCleintsDataToFile(ClientFileName, vClients);
                cout << "\nDone, Successfully. New balance is : " << Client.AccountBalance << endl;

                return true;
            }
        }
    }

    return false;
}

enum enMainMenueOptions { QuickWithdraw = 1, NormalWithdraw = 2, Deposit = 3, CheckBalance = 4, Logout = 5 };

short ReadQuickWithdrawOption()
{
    short Choice = 0;
    while (Choice < 1 || Choice > 9)
    {
        cout << "Choose what to do from [1] to [9]: ";
        cin >> Choice;
    }
    return Choice;
}

short GetQuickWithdrawAmount(short Option)
{
    switch (Option)
    {
    case 1:
        return 20;
    case 2:
        return 50;
    case 3:
        return 100;
    case 4:
        return 200;
    case 5:
        return 400;
    case 6:
        return 600;
    case 7:
        return 800;
    case 8:
        return 1000;
    default:
        return 0;
    }
}

void  PreformeQuickWithdrawOption(short QuickWithdrawOption)
{

    if (QuickWithdrawOption == 9) // Exit
        return;

    double WithdrawBalance = GetQuickWithdrawAmount(QuickWithdrawOption);

    if (WithdrawBalance > CurrentClient.AccountBalance)
    {
        cout << "\nThe amount exceeds your balance, make another choice.\n";
        cout << "Press Anykey to continue...";
        system("pause>0");

        ShowQuickWithdrawScreen(); 
        return;
    }
    vector <sClient> vClients = LoadCleintsDataFromFile(ClientFileName);
    DepositBalancyToClientByAccountNumber(CurrentClient.AccountNumber, WithdrawBalance * -1, vClients);
    CurrentClient.AccountBalance -= WithdrawBalance;
}

void ShowQuickWithdrawScreen()
{
    system("cls");
    cout << "\n===========================================\n";
    cout << "\tQuick Withdraw Screen\n";
    cout << "===========================================\n";
    cout << "\t[1] 20  \t [2] 50\n";
    cout << "\t[3] 100 \t [4] 200\n";
    cout << "\t[5] 400 \t [6] 600\n";
    cout << "\t[7] 800 \t [8] 1000\n";
    cout << "\t[9] Exit\n";
    cout << "===========================================\n";
    cout << "Your Balance is : " << CurrentClient.AccountBalance << endl;

    PreformeQuickWithdrawOption(ReadQuickWithdrawOption());
}

double ReadWithdrawalAmount()
{
    int Amount = 0;
    do
    {
        cout << "Enter amount Multiply of 5's.?";
        cin >> Amount;

    } while (Amount % 5 !=0);
    return Amount;
}

void PreformeNormalQuickWithdrawOption()
{
    double WithdrawAmount = ReadWithdrawalAmount();

    if (WithdrawAmount > CurrentClient.AccountBalance)
    {
        cout << "\nThe amount exceeds your balance, make another choice.\n";
        cout << "Press Anykey to continue..."; 
        system("pause>0");
        ShowNormalWithdrawScreen();
        return;
    }

    vector <sClient> vClients = LoadCleintsDataFromFile(ClientFileName);
    DepositBalancyToClientByAccountNumber(CurrentClient.AccountNumber, WithdrawAmount * -1, vClients);
    CurrentClient.AccountBalance -= WithdrawAmount;
}

void ShowNormalWithdrawScreen()
{
    system("cls");
    cout << "\n===========================================\n";
    cout << "\tNormal Quick Withdraw Screen\n";
    cout << "===========================================\n";

    PreformeNormalQuickWithdrawOption();
}

double ReadDepositAmount()
{
    double Amount = 0;

    cout << "Please enter a positive Deposit amount ?";
    cin >> Amount;

    while (Amount <= 0)
    {
        cout << "Please enter a positive Deposit amount ?";
        cin >> Amount;
    }
    return Amount;
}

void PreformeDepositOption()
{
    double DepositAmount = ReadDepositAmount();

    vector <sClient> vClients = LoadCleintsDataFromFile(ClientFileName);
    DepositBalancyToClientByAccountNumber(CurrentClient.AccountNumber, DepositAmount, vClients);
    CurrentClient.AccountBalance += DepositAmount;
}

void ShowDepositScreen()
{
    system("cls");
    cout << "\n===========================================\n";
    cout << "\t\Deposit Screen\n";
    cout << "===========================================\n";

    PreformeDepositOption();
}

void ShowCheckBalanceScreen()
{
    system("cls");
    cout << "\n===========================================\n";
    cout << "\tCheck Balance Screen\n";
    cout << "===========================================\n";
    cout << "Your Total Balance is : " << CurrentClient.AccountBalance << endl;
}

void GoBackToMainMenu()
{
    cout << "\n\nPress any key to go back to Main Menue...";
    system("pause>0");
    ShowMainMenue();
}

short ReadMainMenueOptions()
{
    short Option = 0;

    cout << "Choose what do you want to do ? [1 to 5]?";
    cin >> Option;

    return Option;
}

void PerformeMainMenueOptions(enMainMenueOptions MainMenueOptions)
{
    switch (MainMenueOptions)
    {
    case enMainMenueOptions::QuickWithdraw:
        system("cls");
        ShowQuickWithdrawScreen();
        GoBackToMainMenu();
        break;

    case enMainMenueOptions::NormalWithdraw:
        system("cls");
        ShowNormalWithdrawScreen();
        GoBackToMainMenu();
        break;

    case enMainMenueOptions::Deposit:
        system("cls");
        ShowDepositScreen();
        GoBackToMainMenu();
        break;

    case enMainMenueOptions::CheckBalance:
        system("cls");
        ShowCheckBalanceScreen();
        GoBackToMainMenu();
        break;

    case enMainMenueOptions::Logout:
        system("cls");
        Login();
        break;
    }
}

void ShowMainMenue()
{
    system("cls");
    cout << "\n===========================================\n";
    cout << "\t\tMain Menue Screen\n";
    cout << "===========================================\n";
    cout << "\t[1] Quick Withdraw.\n";
    cout << "\t[2] Normal Withdraw.\n";
    cout << "\t[3] Deposit.\n";
    cout << "\t[4] Check Balance.\n";
    cout << "\t[5] Logout.\n";
    cout << "===========================================\n";
    PerformeMainMenueOptions((enMainMenueOptions)ReadMainMenueOptions());
}

bool FindClientByAccountNumberAndPinCode(string AccountNumber, string PinCode, sClient &Client)
{
    vector <sClient> vClients = LoadCleintsDataFromFile(ClientFileName);
    for (sClient& C : vClients)
    {
        if (C.AccountNumber == AccountNumber && C.PinCode == PinCode)
        {
            Client = C;
            return true;
        }
    }
    return false;
}

bool LoadClientInfo(string AccountNumber, string PinCode)
{
    if (FindClientByAccountNumberAndPinCode(AccountNumber, PinCode,CurrentClient))
        return true;
    else
        return false;
}

void Login()
{
    bool LoginFail = false;
    string AccountNumber, PinCode;
    do
    {
        system("cls");
        cout << "\n-----------------------------------\n";
        cout << "\tLogin Screen";
        cout << "\n-----------------------------------\n";

        if (LoginFail)
            cout << "Invalid Account Number/PinCode !\n";

        cout << "\nEnter Account Number :";
        cin >> AccountNumber;

        cout << "\nEnter PinCode :";
        cin >> PinCode;

        LoginFail = !LoadClientInfo(AccountNumber, PinCode);

    } while (LoginFail);

    ShowMainMenue();
}

int main()
{
    Login();
    system("pause > 0");

    return 0;
}