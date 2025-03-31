
```
require './object_oriented/drink'
require './object_oriented/coin'
require './object_oriented/stock'

class VendingMachine

  def initialize
    @stock_of_coke = Stock.new(5) # コーラの在庫数 stock.rbからStockクラスを呼び出してる
    @stock_of_diet_coke = Stock.new(5) # ダイエットコーラの在庫数 stock.rbからStockクラスから呼び出している
    @stock_of_tea = Stock.new(5) # お茶の在庫数
    @number_of_100yen = [Coin::ONE_HUNDRED] * 10 # 100円玉の在庫 coin.rbから100円玉の在庫数を調べている
    @change = [] # お釣り
  end

  def buy(payment, kind_of_drink)
    # 100円と500円だけ受け付ける
    if payment != Coin::ONE_HUNDRED && payment != Coin::FIVE_HUNDRED
      # @change += payment
      @change.push(payment)
      return nil
    end

    # if kind_of_drink == DrinkType::COKE && @quantity_of_coke == 0
    if kind_of_drink == DrinkType::COKE && @stock_of_coke.quantity == 0
      @change.push(payment)
      # @change += payment
      return nil
    # elsif kind_of_drink == DrinkType::DIET_COKE && @quantity_of_diet_coke == 0 then
     elsif kind_of_drink == DrinkType::DIET_COKE && @stock_of_diet_coke.quantity == 0 then
      @change.push(payment)
      # @change += payment
      return nil
    #  elsif kind_of_drink == DrinkType::TEA && @quantity_of_tea == 0 then
      elsif kind_of_drink == DrinkType::TEA && @stock_of_tea.quantity == 0 then
      @change.push(payment)
      # @change += payment
      return nil
    end

    # 釣り銭不足
    # if payment == Coin::FIVE_HUNDRED && @number_of_100yen < 4
    if payment ==  Coin::FIVE_HUNDRED && @number_of_100yen.length < 4
      @change.push(payment)
      return nil
    end

    if payment == Coin::ONE_HUNDRED
      @number_of_100yen.push(payment)
    elsif payment == Coin::FIVE_HUNDRED then
      # 400円のお釣り
      # @change += (payment - Coin::ONE_HUNDRED)
      @change = @change.concat(calculate_change)
      # 100円玉を釣り銭に使える
      # @number_of_100yen -= (payment - ONE_HUNDRED) / ONE_HUNDRED
    end

    # if kind_of_drink == Drink::COKE
    if kind_of_drink == DrinkType::COKE
      # @quantity_of_coke -= 1
      @stock_of_coke.decrement
    # elsif kind_of_drink == Drink::DIET_COKE then
      # @quantity_of_diet_coke -= 1
    elsif kind_of_drink == DrinkType::DIET_COKE then
      @stock_of_diet_coke.decrement
    else
      # @quantity_of_tea -= 1
      @stock_of_tea.decrement
    end

    Drink.new(kind_of_drink)
  end

  def refund
    result = @change
    @change = 0
    result
  end

  def calculate_change
    @number_of_100yen.slice!(0, 4)
    [Coin::ONE_HUNDRED] * 4
  end
end
```