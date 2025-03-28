TestTaskManager中需要实现以下的功能：
1. 根据我们项目中的ChannelMapping集合内部的信息为每一个测试点位创建测试Task(需要区分AI、AO、DI、DO)创建不同的测试任务，且将测试任务通过一些方法来管理开始、暂停、添加、删除等操作，这个任务的内部我们稍后再写入具体实现，你先帮我创建好ITestTaskManager的接口内的方法。要求如下：
	1. 测试任务之间不能相互干扰，且传入的和PLC的通讯实例对于测试PLC和被测PLC都是单例的。
	2. 对于单点测试任务需要能同步开展，使用多线程来实现。
	3. 线程内部由于需要按照流程进行测试所以需要等待固定的时间接收测试结果。所以测试线程不能阻塞主线程。一般一个AI、AO点位需要测试15秒左右
	4. 我考虑如果所有的点位都单独开一个线程那么线程池肯定不够，我希望你帮我设计好Task的处理方式，能保证所有单点测试同步开展并且不会造成频繁的上下文切换从而影响性能

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.ComponentModel;

namespace FatFullVersion.Models
{
    /// <summary>
    /// 测试结果类，用于存储测试的结果数据
    /// </summary>
    public class TestResult : INotifyPropertyChanged
    {
        private string _variableName;
        /// <summary>
        /// 变量名称
        /// </summary>
        public string VariableName
        {
            get { return _variableName; }
            set
            {
                if (_variableName != value)
                {
                    _variableName = value;
                    OnPropertyChanged(nameof(VariableName));
                }
            }
        }

        private string _testPlcChannel;
        /// <summary>
        /// 测试PLC通道标识
        /// </summary>
        public string TestPlcChannel
        {
            get { return _testPlcChannel; }
            set
            {
                if (_testPlcChannel != value)
                {
                    _testPlcChannel = value;
                    OnPropertyChanged(nameof(TestPlcChannel));
                }
            }
        }

        private DateTime _startTime;
        /// <summary>
        /// 测试开始时间
        /// </summary>
        public DateTime StartTime
        {
            get { return _startTime; }
            set
            {
                if (_startTime != value)
                {
                    _startTime = value;
                    OnPropertyChanged(nameof(StartTime));
                }
            }
        }

        private DateTime _endTime;
        /// <summary>
        /// 测试结束时间
        /// </summary>
        public DateTime EndTime
        {
            get { return _endTime; }
            set
            {
                if (_endTime != value)
                {
                    _endTime = value;
                    OnPropertyChanged(nameof(EndTime));
                    // 当结束时间设置时，自动计算测试持续时间
                    OnPropertyChanged(nameof(TestDuration));
                }
            }
        }

        /// <summary>
        /// 测试持续时间（秒）
        /// </summary>
        public double TestDuration
        {
            get
            {
                if (_endTime > DateTime.MinValue && _startTime > DateTime.MinValue)
                {
                    return (_endTime - _startTime).TotalSeconds;
                }
                return 0;
            }
        }

        private double _expectedValue;
        /// <summary>
        /// 期望值
        /// </summary>
        public double ExpectedValue
        {
            get { return _expectedValue; }
            set
            {
                if (_expectedValue != value)
                {
                    _expectedValue = value;
                    OnPropertyChanged(nameof(ExpectedValue));
                }
            }
        }

        private double _actualValue;
        /// <summary>
        /// 实际值
        /// </summary>
        public double ActualValue
        {
            get { return _actualValue; }
            set
            {
                if (_actualValue != value)
                {
                    _actualValue = value;
                    OnPropertyChanged(nameof(ActualValue));
                    // 当实际值设置时，自动计算偏差值
                    OnPropertyChanged(nameof(Deviation));
                    OnPropertyChanged(nameof(DeviationPercent));
                }
            }
        }

        /// <summary>
        /// 偏差值
        /// </summary>
        public double Deviation
        {
            get { return ActualValue - ExpectedValue; }
        }

        /// <summary>
        /// 偏差百分比
        /// </summary>
        public double DeviationPercent
        {
            get
            {
                if (ExpectedValue != 0)
                {
                    return (Deviation / ExpectedValue) * 100;
                }
                return 0;
            }
        }

        private string _status;
        /// <summary>
        /// 测试状态（通过/失败/取消等）
        /// </summary>
        public string Status
        {
            get { return _status; }
            set
            {
                if (_status != value)
                {
                    _status = value;
                    OnPropertyChanged(nameof(Status));
                }
            }
        }

        private string _errorMessage;
        /// <summary>
        /// 错误信息
        /// </summary>
        public string ErrorMessage
        {
            get { return _errorMessage; }
            set
            {
                if (_errorMessage != value)
                {
                    _errorMessage = value;
                    OnPropertyChanged(nameof(ErrorMessage));
                }
            }
        }

        private string _batchName;
        /// <summary>
        /// 测试批次名称
        /// </summary>
        public string BatchName
        {
            get { return _batchName; }
            set
            {
                if (_batchName != value)
                {
                    _batchName = value;
                    OnPropertyChanged(nameof(BatchName));
                }
            }
        }

        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged(string propertyName)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
} 

```


我现在需要你帮我完成以下的这些工作：
修改批次选择弹窗中绑定的相关数据：
首先我先定义需要的列如下：
批次名称、测试项点数、创建时间、初次测试时间、最终测试时间、测试状态
其次我需要用户在导入点表后，系统生成默认的测试批次的同时将批次信息同步的更新到这个批次选择窗口中，其中
创建时间为自动生成点表的时间后续需要能通过其他的时间进行修改
测试项点数为当前批次4中类型点数的总和
初次测试时间为同组内测试时间最早初次的测试时间为准，最终测试时间为同组内最终测试时间最晚的为准。测试状态已完成、取消、进行中、未开始这四种状态为准。请你在修改前端界面(DataEditView)的同时帮我修改后台逻辑中涉及到的数据模型和处理逻辑


现在让我们来解决下面的这些问题
1、批次选择窗口中的批次状态没有根据批次内的测试点状态实时更新
2、测试队列显示区域
3、AO点位的低低报、低报、高报、高高报在测试的时候不涉及调整为NaN
4、AI点位在测试的时候高报高高报区域点击确认通过后对于的测试结果中的高报反馈状态、高高报反馈状态的结果都设置为已通过。低报及低低报区域也同理
5、测试结果中的内容设置为追加模式，对于同样的内容替换，不同的内容追加。结果尽可能的详细
6、


