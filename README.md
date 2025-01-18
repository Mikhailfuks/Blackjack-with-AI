using System;
using System.Collections.Generic;

public class Program
{
    static void Main(string[] args)
    {
        BlackjackGame game = new BlackjackGame();
        game.Start();
    }
}

public class BlackjackGame
{
    private Deck deck;
    private Player player;
    private Player dealer;

    public BlackjackGame()
    {
        deck = new Deck();
        player = new Player("Игрок");
        dealer = new Player("Дилер");
    }

    public void Start()
    {
        deck.Shuffle();
        InitialDeal();

        // Игровой цикл
        while (true)
        {
            Console.Clear();
            DisplayGameStatus();

            if (player.Score == 21)
            {
                Console.WriteLine("Вы выиграли! У вас блэкджек!");
                break;
            }
            else if (player.Score > 21)
            {
                Console.WriteLine("Вы проиграли! Перебор!");
                break;
            }

            Console.Write("Хотите взять карту? (y/n): ");
            string choice = Console.ReadLine();
            if (choice.ToLower() == "y")
            {
                player.AddCard(deck.DealCard());
            }
            else
            {
                // Дилер играет
                while (dealer.Score < 17)
                {
                    dealer.AddCard(deck.DealCard());
                }

                // Определение победителя
                DetermineWinner();
                break;
            }
        }

        Console.WriteLine("Спасибо за игру!");
    }

    private void InitialDeal()
    {
        for (int i = 0; i < 2; i++)
        {
            player.AddCard(deck.DealCard());
            dealer.AddCard(deck.DealCard());
        }
    }

    private void DisplayGameStatus()
    {
        Console.WriteLine($"{dealer.Name}'s score: {dealer.Score} (одна карта скрыта)");
        Console.WriteLine($"{player.Name}'s score: {player.Score}");
        Console.WriteLine("Ваши карты: ");
        player.DisplayCards();
        Console.WriteLine();
        
        Console.WriteLine($"{dealer.Name}'s cards: ");
        dealer.DisplayCards(revealAll: true);
        Console.WriteLine();
    }

    private void DetermineWinner()
    {
        Console.Clear();
        DisplayGameStatus();

        if (dealer.Score > 21 || player.Score > dealer.Score)
        {
            Console.WriteLine("Вы выиграли!");
        }
        else if (player.Score < dealer.Score)
        {
            Console.WriteLine("Вы проиграли.");
        }
        else
        {
            Console.WriteLine("Ничья.");
        }
    }
}

public class Player
{
    public string Name { get; }
    private List<Card> hand;

    public Player(string name)
    {
        Name = name;
        hand = new List<Card>();
    }

    public int Score
    {
        get
        {
            int score = 0;
            int acesCount = 0;
            foreach (Card card in hand)
            {
                score += card.Value;
                if (card.Rank == Rank.Ace)
                {
                    acesCount++;
                }
            }

            while (score > 21 && acesCount > 0)
            {
                score -= 10; // Считаем туз как 1
                acesCount--;
            }

            return score;
        }
    }

    public void AddCard(Card card)
    {
        hand.Add(card);
    }


    public void DisplayCards(bool revealAll = false)
    {
        for (int i = 0; i < hand.Count; i++)
        {
            Console.Write(hand[i].ToString() + " ");
        }
        Console.WriteLine();
    }
}

public class Card
{
    public Rank Rank { get; }
    public Suit Suit { get; }
    public int Value => Rank == Rank.Ace ? 11 : (int)Rank > 10 ? 10 : (int)Rank;

    public Card(Rank rank, Suit suit)
    {
        Rank = rank;
        Suit = suit;
    }

    public override string ToString()
    {
        return $"{Rank} of {Suit}";
    }
}

public enum Rank
{
    Ace = 1,
    Two,
    Three,
    Four,
    Five,
    Six,
    Seven,
    Eight,
    Nine,
    Ten,
    Jack,
    Queen,
    King
}

public enum Suit
{
    Hearts,
    Diamonds,
    Clubs,
    Spades
}

public class Deck
{
    private List<Card> cards;
    private Random random;

    public Deck()
    {
        cards = new List<Card>();
        foreach (Suit suit in Enum.GetValues(typeof(Suit)))
        {
            foreach (Rank rank in Enum.GetValues(typeof(Rank)))
            {
                cards.Add(new Card(rank, suit));
            }
        }
        random = new Random();
    }

    public void Shuffle()
    {
        int n = cards.Count;
        while (n > 1)
        {
            int k = random.Next(n);
            n--;
            var card = cards[n];
            cards[n] = cards[k];
            cards[k] = card;
        }
    }

    public Card DealCard()
    {
        if (cards.Count == 0) throw new InvalidOperationException("Нет карт для раздачи.");
        Card dealtCard = cards[cards.Count - 1];
        cards.RemoveAt(cards.Count - 1);
        return dealtCard;
    }
}
