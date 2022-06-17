**1.** в запросе hw_triggers.sql ошибка  
нет ; в строке SET search_path = pract_functions, publ

**2.** Создаем проведуру для тригера  
*create or replace function good_sum_mart() returns trigger as  
    $$  
    declare good_name varchar(63);  
    begin  
           
    IF OLD is null THEN  
        select   
            case when good_sum_mart.good_name is null then  
               goods.good_name  
            else  
                ''  
            end  
        from   
            pract_functions.sales as sales  
            inner join pract_functions.goods as goods  
                   on sales.good_id = goods.goods_id  
            left outer join pract_functions.good_sum_mart as good_sum_mart  
                on goods.good_name = good_sum_mart.good_name  
        where  
            sales.good_id = new.good_id  
        into good_name;  
    ELSE  
        select '' into good_name;  
    END IF;  
    
    IF good_name <> '' THEN  
        insert into pract_functions.good_sum_mart select good_name, 0;  
    END IF;  
        
    update pract_functions.good_sum_mart  
        SET sum_sale = sum_sale  
            + CASE WHEN NEW is NULL THEN 0 ELSE NEW.sales_qty * goods.good_price END  
            - CASE WHEN OLD is NULL THEN 0 ELSE OLD.sales_qty * goods.good_price END  
        FROM  
            pract_functions.goods as goods  
        WHERE  
            pract_functions.good_sum_mart.good_name = goods.good_name  
            and goods.goods_id = CASE WHEN NEW is NULL THEN OLD.good_id ELSE NEW.good_id END;  
       
    return null;  
       
    end  
    $$  
    language plpgsql  
    ;*  
