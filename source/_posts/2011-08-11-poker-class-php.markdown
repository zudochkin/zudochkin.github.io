---
author: admin
comments: true
date: 2011-08-11 09:52:41+00:00
layout: post
slug: poker-class-php
title: Покерный класс на PHP
wordpress_id: 752
categories:
- OOP
- php
- алгоритмы
- программирование
tags:
- OOP
- php
- poker
---


Приветствую тебя, разработчик или случайно зашедший на огонег в этот уютный технический бложек. Сегодня речь пойдет о несколько математической задаче, хоть и немного там математики, да и она понятна школьнику 5го класса. Сегодня мы научимся распознавать комбинации в Техасском холдеме, именно в такую разновидность покера я играю (играл). Т.к. это очень простой класс на PHP, он не будет иметь определять старшинство одинаковых по названию комбинаций, но разнящихся по номиналу, к примеру стрит до 10 и стрит до 8 будут трактоваться этим классом, как просто стрит. В Техасском холдеме всего 10 комбинаций и мы с вами пройдемся от самой старшей из них - флеш рояль до самой младшей - старшая карта.
<!-- more -->


###### Флеш рояль


	Высшая кобинация в покере - это стрит (стрейт) флеш до туза.
[cc lang="php"]
protected function _isRoyalFlush()
{
	return ($this->_isFlush() && $this->_isStraight() && array_search(14, $this->_ranks) && array_search('13', $this->_ranks));
}
[/cc]

![poker-class-php](http://vredniy.ru/wp-content/uploads/2011/08/poker-class-php-300x214.jpg)Немного костыльный способ проверки на флеш рояль: проверяем на наличие стрита, флеша и чтобы наличствовали туз и кароль.




###### Каре или Quad


Тут проще некуда, нужно наличие четырех одинаковых карт. 

[cc lang="php"]
protected function _isQuad()
{

	$test = $this->_ranks;

	$uniqueElementsCount = array();

	foreach($this->_ranks as $key => $card) {
		$test = $this->_ranks;
		unset($test[$key]);
		$uniqueElementsCount[] = count(array_unique($test));
	}


	return 1 === min($uniqueElementsCount);
}
[/cc]



###### Стрит флеш


Тот же самый флешрояль, только необязательно, чтобы стрит был до туза (10-валет-дама-кароль-туз).
[cc lang="php"]
protected function _isStraightFlush()
{
	return ($this->_isFlush() && $this->_isStraight());
}
[/cc]



###### Фул хаус



Это когда три карты одного достоинства и две другого. Не самая удачная реализация :)
[cc lang="php"]
protected function _isFullHouse()
{
	$ranks = $this->_ranks;
	sort($ranks);

	if ((($ranks[0] == $ranks[1] && $ranks[1] == $ranks[2]) && ($ranks[3] == $ranks[4])) // 1=2=3 and 4=5
			|| ($ranks[0] == $ranks[1]) && ($ranks[2] == $ranks[3] and $ranks[3] == $ranks[4])) // 1=2 and 3=4=5
		return true;

	return false;
    }
[/cc]



###### Флеш


Все карты в комбинации должны быть одной масти. Проверяет все очень просто: сортируется массив с мастями и проверяется на равенство первая и последняя карты.

[cc lang="php"]
protected function _isFlush()
{
	$suits = $this->_suits;
	sort($suits);
	if ($suits[0] === $suits[4])
		return true;

	return false;
}
[/cc]



###### Стрит


Вне зависимости от масти, достоинство карт должно быть по старшенству (5 подряд). Из-за того, что туз может в зависимости от ситуации быть и самой старшей, и самой младшей картой, пришлось сделать две проверки.

[cc lang="php"]
protected function _isStraight()
{

	$ranks = $this->_ranks;
	sort($ranks);

	// if Ace is low card in straight
	if ($key = array_search(14, $ranks)) {
		$tempRanks = $ranks;
		unset($tempRanks[$key]);
		if (array(2, 3, 4, 5) == $tempRanks) {
			return true;
		}
		unset($tempRanks);
	}

	// if Ace is high card - default algorithm
	$min = $ranks[0];
	foreach($ranks as $key => $value) {
		$ranks[$key] -= $min;
		if ($key != $ranks[$key])
			return false;
	}
	return true;
}
[/cc]



###### Сет, трипс или тройка


Нужно наличие трех одинаковых карт. Алгоритм опять же далек от совершенства, но он выполняет свою работу.

[cc lang="php"]
protected function _isThreeOfKind()
{
	$ranks = $this->_ranks;
	sort($ranks);

	if (($ranks[0] == $ranks[1] and $ranks[1] == $ranks[2])
			|| ($ranks[1] == $ranks[2] and $ranks[2] == $ranks[3])
			|| ($ranks[2] == $ranks[3] and $ranks[3] == $ranks[4]))
		return true;
	return false;
}
[/cc]



###### Две пары


Из название алгоритма должно быть понятно что к чему.

[cc lang="php"]
protected function _isTwoPairs()
{
	$ranks = $this->_ranks;

	foreach($ranks as $key => $rank) {
		$testRanks = $ranks;
		unset($testRanks[$key]);
		sort($testRanks);
		if (($testRanks[0] == $testRanks[1])
				and ($testRanks[2] == $testRanks[3])
				and ($testRanks[0] != $testRanks[2])
				// exclude full house
				and ($testRanks[0] != $ranks[$key])
				and ($testRanks[2] != $ranks[$key])
		)
			return true;
	}
	return false;
}
[/cc]



###### Пара


Количество уникальных карт из пяти должно равняться четырем, если опять же я ничего не перепутал :).
[cc lang="php"]
protected function _isPair()
{
	return 4 === count(array_unique($this->_ranks));
}
[/cc]



###### Старшая карта


Последний абзаци больше для красоты, потому что если ни одна комбинация не подошла, значит комбинация - старшая карта.
[cc lang="php"]
protected function _isHighCard()
{
	return true;
}
[/cc]




###### Исходный код класса для опеределения покерных комбинаций на PHP


[cc lang="php"]
_cards = $cards;

        $rank = null;

        foreach($cards as $card) {
            switch (strtolower($card['rank'])) {
                case 't':
                    $rank = 10;
                    break;
                case 'j':
                    $rank = '11';
                    break;
                case 'q':
                    $rank = '12';
                    break;
                case 'k':
                    $rank = '13';
                    break;
                case 'a':
                    $rank = '14';
                    break;
                    ;
                default:
                    $rank = $card['rank'];
                    break;
            }
            $this->_ranks[] = $rank;
            $this->_suits[] = $card['suit'];
        }
    }

    public function checkCombination()
    {
        // is Royal Flush
        echo $this->_isRoyalFlush() ? 'royal flush' : 'not royal flush';
        echo '  
';

        // is Quad (four of kind)
        echo $this->_isQuad() ? 'quad' : 'not quad';
        echo '  
';

        // is StraightFlush
        echo $this->_isStraightFlush() ? 'straight flush' : 'not straight flush';
        echo '  
';

        // is Full House
        echo $this->_isFullHouse() ? 'full house' : 'not full house';
        echo '  
';

        // is Flush
        echo $this->_isFlush() ? 'flush' : 'not flush';
        echo '  
';

        // is Straight
        echo $this->_isStraight() ? 'straight' : 'not straigt';
        echo '  
';

        // is Three of Kind
        echo $this->_isThreeOfKind() ? 'three of kind' : 'not three of kind';
        echo '  
';

        // is Two Pairs
        echo $this->_isTwoPairs() ? 'two pairs' : 'not two pairs';
        echo '  
';

        // is one Pair
        echo $this->_isPair() ? 'pair' : 'not pair';
        echo '  
';

        // is High Card
        echo $this->_isHighCard() ? 'high card' : 'not high card';
        echo '  
';
    }

    /**
     * chech that combination is flush and straight and contains ace and king (exclude straight flush "ace-5")
     *
     * @return bool
     */
    protected function _isRoyalFlush()
    {
        return ($this->_isFlush() && $this->_isStraight() && array_search(14, $this->_ranks) && array_search('13', $this->_ranks));
    }

    /**
     * check combination is quad (four of kind)
     *
     * @return bool
     */
    protected function _isQuad()
    {

        $test = $this->_ranks;

        $uniqueElementsCount = array();

        foreach($this->_ranks as $key => $card) {
            $test = $this->_ranks;
            unset($test[$key]);
            $uniqueElementsCount[] = count(array_unique($test));
        }


        return 1 === min($uniqueElementsCount);
    }

    /**
     * is a straight and a flush?
     *
     * @return bool
     */
    protected function _isStraightFlush()
    {
        return ($this->_isFlush() && $this->_isStraight());
    }

    /**
     * is full house?
     * (1=2=3 and 4=5) or (1=2 and 3=4=5)
     *
     * @return bool
     */
    protected function _isFullHouse()
    {
        $ranks = $this->_ranks;
        sort($ranks);

        if ((($ranks[0] == $ranks[1] && $ranks[1] == $ranks[2]) && ($ranks[3] == $ranks[4])) // 1=2=3 and 4=5
                || ($ranks[0] == $ranks[1]) && ($ranks[2] == $ranks[3] and $ranks[3] == $ranks[4])) // 1=2 and 3=4=5
            return true;

        return false;
    }

    /**
     * is flush?
     *
     * @return bool
     */
    protected function _isFlush()
    {
        $suits = $this->_suits;
        sort($suits);
        if ($suits[0] === $suits[4])
            return true;

        return false;
    }

    /**
     * check straight. 2 attempt, 'cause ace may be high card, or low.
     *
     * @return bool
     */
    protected function _isStraight()
    {

        $ranks = $this->_ranks;
        sort($ranks);

        // if Ace is low card in straight
        if ($key = array_search(14, $ranks)) {
            $tempRanks = $ranks;
            unset($tempRanks[$key]);
            if (array(2, 3, 4, 5) == $tempRanks) {
                return true;
            }
            unset($tempRanks);
        }

        // if Ace is high card - default algorithm
        $min = $ranks[0];
        foreach($ranks as $key => $value) {
            $ranks[$key] -= $min;
            if ($key != $ranks[$key])
                return false;
        }
        return true;
    }

    /**
     * is Three of kind
     *
     * @return bool
     */
    protected function _isThreeOfKind()
    {
        $ranks = $this->_ranks;
        sort($ranks);

        if (($ranks[0] == $ranks[1] and $ranks[1] == $ranks[2])
                || ($ranks[1] == $ranks[2] and $ranks[2] == $ranks[3])
                || ($ranks[2] == $ranks[3] and $ranks[3] == $ranks[4]))
            return true;
        return false;
    }

    /**
     * is two pairs
     *
     * @return bool
     */
    protected function _isTwoPairs()
    {
        $ranks = $this->_ranks;

        foreach($ranks as $key => $rank) {
            $testRanks = $ranks;
            unset($testRanks[$key]);
            sort($testRanks);
            if (($testRanks[0] == $testRanks[1])
                    and ($testRanks[2] == $testRanks[3])
                    and ($testRanks[0] != $testRanks[2])
                    // exclude full house
                    and ($testRanks[0] != $ranks[$key])
                    and ($testRanks[2] != $ranks[$key])
            )
                return true;
        }
        return false;
    }

    /**
     * is single Pair
     *
     * @return bool
     */
    protected function _isPair()
    {
        return 4 === count(array_unique($this->_ranks));
    }

    /**
     * is High Card
     *
     * @return bool true
     */
    protected function _isHighCard()
    {
        return true;
    }

}

[/cc]


###### Небольшой пример использования


[cc lang="php"]
's', 'rank' => '3'),
    array('suit' => 's', 'rank' => '7'),
    array('suit' => 's', 'rank' => 'a'),
    array('suit' => 's', 'rank' => 't'),
    array('suit' => 's', 'rank' => 't')
);

$poker = new Poker($cards);

$poker->checkCombination();
[/cc]
Комбинация из 5 карт для данного класса задается массивом из 5 элементов, которые в свою очередь явлеются ассоциативными массивами (suit - это масть, rank - достоинство карты). 
Эпилог
	Данный класс можно немного доработать, чтобы вместо название комбинаций, он выводил какой-нибудь балл комбинации, чтобы имелась возможность сравнивать две одинаковых комбинации, но это по желанию и выходит за пределы данной заметки. Также вы можете допилить проверку на корректность заданной комбинации. На этом на сегодня все.	Удачи вам в любых начинаниях и продолжениях.
