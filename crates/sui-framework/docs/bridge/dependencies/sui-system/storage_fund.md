
<a name="0x3_storage_fund"></a>

# Module `0x3::storage_fund`



-  [Struct `StorageFund`](#0x3_storage_fund_StorageFund)
-  [Function `new`](#0x3_storage_fund_new)
-  [Function `advance_epoch`](#0x3_storage_fund_advance_epoch)
-  [Function `total_object_storage_rebates`](#0x3_storage_fund_total_object_storage_rebates)
-  [Function `total_balance`](#0x3_storage_fund_total_balance)


<pre><code><b>use</b> <a href="../../dependencies/sui-framework/balance.md#0x2_balance">0x2::balance</a>;
<b>use</b> <a href="../../dependencies/sui-framework/sui.md#0x2_sui">0x2::sui</a>;
</code></pre>



<a name="0x3_storage_fund_StorageFund"></a>

## Struct `StorageFund`



<pre><code><b>struct</b> <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_StorageFund">StorageFund</a> <b>has</b> store
</code></pre>



<details>
<summary>Fields</summary>


<dl>
<dt>
<code>total_object_storage_rebates: <a href="../../dependencies/sui-framework/balance.md#0x2_balance_Balance">balance::Balance</a>&lt;<a href="../../dependencies/sui-framework/sui.md#0x2_sui_SUI">sui::SUI</a>&gt;</code>
</dt>
<dd>

</dd>
<dt>
<code>non_refundable_balance: <a href="../../dependencies/sui-framework/balance.md#0x2_balance_Balance">balance::Balance</a>&lt;<a href="../../dependencies/sui-framework/sui.md#0x2_sui_SUI">sui::SUI</a>&gt;</code>
</dt>
<dd>

</dd>
</dl>


</details>

<a name="0x3_storage_fund_new"></a>

## Function `new`



<pre><code><b>public</b>(<b>friend</b>) <b>fun</b> <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_new">new</a>(initial_fund: <a href="../../dependencies/sui-framework/balance.md#0x2_balance_Balance">balance::Balance</a>&lt;<a href="../../dependencies/sui-framework/sui.md#0x2_sui_SUI">sui::SUI</a>&gt;): <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_StorageFund">storage_fund::StorageFund</a>
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b>(<b>friend</b>) <b>fun</b> <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_new">new</a>(initial_fund: Balance&lt;SUI&gt;) : <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_StorageFund">StorageFund</a> {
    <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_StorageFund">StorageFund</a> {
        // At the beginning there's no <a href="../../dependencies/sui-framework/object.md#0x2_object">object</a> in the storage yet
        total_object_storage_rebates: <a href="../../dependencies/sui-framework/balance.md#0x2_balance_zero">balance::zero</a>(),
        non_refundable_balance: initial_fund,
    }
}
</code></pre>



</details>

<a name="0x3_storage_fund_advance_epoch"></a>

## Function `advance_epoch`



<pre><code><b>public</b>(<b>friend</b>) <b>fun</b> <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_advance_epoch">advance_epoch</a>(self: &<b>mut</b> <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_StorageFund">storage_fund::StorageFund</a>, storage_charges: <a href="../../dependencies/sui-framework/balance.md#0x2_balance_Balance">balance::Balance</a>&lt;<a href="../../dependencies/sui-framework/sui.md#0x2_sui_SUI">sui::SUI</a>&gt;, storage_fund_reinvestment: <a href="../../dependencies/sui-framework/balance.md#0x2_balance_Balance">balance::Balance</a>&lt;<a href="../../dependencies/sui-framework/sui.md#0x2_sui_SUI">sui::SUI</a>&gt;, leftover_staking_rewards: <a href="../../dependencies/sui-framework/balance.md#0x2_balance_Balance">balance::Balance</a>&lt;<a href="../../dependencies/sui-framework/sui.md#0x2_sui_SUI">sui::SUI</a>&gt;, storage_rebate_amount: u64, non_refundable_storage_fee_amount: u64): <a href="../../dependencies/sui-framework/balance.md#0x2_balance_Balance">balance::Balance</a>&lt;<a href="../../dependencies/sui-framework/sui.md#0x2_sui_SUI">sui::SUI</a>&gt;
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b>(<b>friend</b>) <b>fun</b> <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_advance_epoch">advance_epoch</a>(
    self: &<b>mut</b> <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_StorageFund">StorageFund</a>,
    storage_charges: Balance&lt;SUI&gt;,
    storage_fund_reinvestment: Balance&lt;SUI&gt;,
    leftover_staking_rewards: Balance&lt;SUI&gt;,
    storage_rebate_amount: u64,
    non_refundable_storage_fee_amount: u64,
) : Balance&lt;SUI&gt; {
    // Both the reinvestment and leftover rewards are not <b>to</b> be refunded so they go <b>to</b> the non-refundable <a href="../../dependencies/sui-framework/balance.md#0x2_balance">balance</a>.
    <a href="../../dependencies/sui-framework/balance.md#0x2_balance_join">balance::join</a>(&<b>mut</b> self.non_refundable_balance, storage_fund_reinvestment);
    <a href="../../dependencies/sui-framework/balance.md#0x2_balance_join">balance::join</a>(&<b>mut</b> self.non_refundable_balance, leftover_staking_rewards);

    // The storage charges for the epoch come from the storage rebate of the new objects created
    // and the new storage rebates of the objects modified during the epoch so we put the charges
    // into `total_object_storage_rebates`.
    <a href="../../dependencies/sui-framework/balance.md#0x2_balance_join">balance::join</a>(&<b>mut</b> self.total_object_storage_rebates, storage_charges);

    // Split out the non-refundable portion of the storage rebate and put it into the non-refundable <a href="../../dependencies/sui-framework/balance.md#0x2_balance">balance</a>.
    <b>let</b> non_refundable_storage_fee = <a href="../../dependencies/sui-framework/balance.md#0x2_balance_split">balance::split</a>(&<b>mut</b> self.total_object_storage_rebates, non_refundable_storage_fee_amount);
    <a href="../../dependencies/sui-framework/balance.md#0x2_balance_join">balance::join</a>(&<b>mut</b> self.non_refundable_balance, non_refundable_storage_fee);

    // `storage_rebates` <b>include</b> the already refunded rebates of deleted objects and <b>old</b> rebates of modified objects and
    // should be taken out of the `total_object_storage_rebates`.
    <b>let</b> storage_rebate = <a href="../../dependencies/sui-framework/balance.md#0x2_balance_split">balance::split</a>(&<b>mut</b> self.total_object_storage_rebates, storage_rebate_amount);

    // The storage rebate <b>has</b> already been returned <b>to</b> individual transaction senders' gas coins
    // so we <b>return</b> the <a href="../../dependencies/sui-framework/balance.md#0x2_balance">balance</a> <b>to</b> be burnt at the very end of epoch change.
    storage_rebate
}
</code></pre>



</details>

<a name="0x3_storage_fund_total_object_storage_rebates"></a>

## Function `total_object_storage_rebates`



<pre><code><b>public</b> <b>fun</b> <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_total_object_storage_rebates">total_object_storage_rebates</a>(self: &<a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_StorageFund">storage_fund::StorageFund</a>): u64
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_total_object_storage_rebates">total_object_storage_rebates</a>(self: &<a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_StorageFund">StorageFund</a>): u64 {
    <a href="../../dependencies/sui-framework/balance.md#0x2_balance_value">balance::value</a>(&self.total_object_storage_rebates)
}
</code></pre>



</details>

<a name="0x3_storage_fund_total_balance"></a>

## Function `total_balance`



<pre><code><b>public</b> <b>fun</b> <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_total_balance">total_balance</a>(self: &<a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_StorageFund">storage_fund::StorageFund</a>): u64
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_total_balance">total_balance</a>(self: &<a href="../../dependencies/sui-system/storage_fund.md#0x3_storage_fund_StorageFund">StorageFund</a>): u64 {
    <a href="../../dependencies/sui-framework/balance.md#0x2_balance_value">balance::value</a>(&self.total_object_storage_rebates) + <a href="../../dependencies/sui-framework/balance.md#0x2_balance_value">balance::value</a>(&self.non_refundable_balance)
}
</code></pre>



</details>